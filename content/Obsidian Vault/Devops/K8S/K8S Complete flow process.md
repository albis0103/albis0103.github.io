
K8S 流量分為**外部進入（North-South）**與**叢集內部（East-West）**兩條路徑。以下以最典型的 Cloud LB → Ingress → Service → Pod 為例。

## 一、外部請求完整路徑

```
Client
  │ ① DNS 解析 → LB 公網 IP 
  ▼
External Load Balancer（雲端 LB / MetalLB）
  │ ② 轉發至 NodePort（或直接到 Pod，視 CNI/LB 整合）
  ▼
Node: kube-proxy 規則（iptables / IPVS / nftables）
  │ ③ DNAT 把封包送到對應 port
  ▼
Ingress Controller Pod（NGINX / Traefik / Envoy）
  | 解析 HTTP request 讀 Host, Path, TLS
  | 路由對應service
  │ ④ TLS 終止；依 Host / Path 比對 Ingress 規則
  ▼
Service（ClusterIP）
  │ ⑤ kube-proxy DNAT：ClusterIP:port → PodIP:targetPort
  ▼
Pod（經 CNI 網路，可能跨 Node）
  │ ⑥ 進入 Pod netns（veth pair）→ Container
  ▼
Application
```

回程封包由 conntrack 記錄的 NAT 狀態反向轉換，原路返回。

## 二、各層細節

**1. DNS 與 LB (Load Balancer)**

- 域名解析到 LB 的 External IP。`Service type=LoadBalancer` 由 cloud-controller-manager 建立 LB，後端為各 Node 的 NodePort（30000–32767）。


**2. externalTrafficPolicy：選機房，判斷要不要跨機器**
：User(`1.2.3.4`) 請求 Node A -Pod，但Pod忙碌中Node A把流量轉發到 Node B相同服務Pod
- `Cluster`（預設）：流量可被轉到其他 Node 的 Pod，需 SNAT(Source NAT)，把 client IP 改成 Node A IP，讓 Node A 回傳 User。
	- note: 三角路由問題：User 發 Node A request , 收到 Node B response
- `Local`：只轉給本機 Pod，保留來源 IP，但可能負載不均；LB health check 會剔除無本機 Pod 的 Node。

**3. kube-proxy（Service 的資料平面）：DNAT把流量給Ingress pod**
https://kubernetes.io/zh-cn/docs/concepts/architecture/
![[Pasted image 20261011162029.png]]
- 監聽 API Server 的 Service / EndpointSlice 變化，在每個 Node 寫入規則：
    - **iptables**：KUBE-SERVICES → KUBE-SVC-xxx → KUBE-SEP-xxx，以 `statistic random` 做機率式負載平衡，O(n) 規則比對。
    - **IPVS**：核心 hash table，O(1) 查找，支援 rr / lc / sh 等演算法。
- 動作是 **DNAT**（Destination NAT），**knob-proxy 透過 iptables or IPVS 規則攔截封包，把目的 `Service IP`改成 真實 `Ingress Controller Pod IP`**
- Cilium eBPF 模式可取代 kube-proxy，在 socket/tc 層直接做轉換。



**4. Ingress**

- Ingress 資源只是規則；Ingress Controller 是實際的 L7 反向代理。
- Controller 通常**直接用 EndpointSlice 的 Pod IP 作為 upstream**，繞過 Service 的 ClusterIP（不經 kube-proxy 第二次 DNAT）。
- Gateway API（Gateway / HTTPRoute）是其後繼設計。

**5. CNI（Pod 網路）**

- 每個 Pod 有獨立 netns，經 veth pair 連接 Node 端的 bridge 或路由。
- 同 Node：經 bridge / 路由直接轉送。
- 跨 Node：
    - Overlay（Flannel VXLAN、Calico IPIP）：封裝後經 Node 網路傳輸。
    - Underlay / BGP（Calico BGP、Cilium native routing）：直接路由，無封裝。

## 三、叢集內部（East-West）

```
Pod A → 查詢 CoreDNS（svc.ns.svc.cluster.local）
      → 取得 ClusterIP
      → 送出封包，Node 上 kube-proxy 規則 DNAT 為 PodIP
      → CNI 轉送至 Pod B
```

- Pod 的 `/etc/resolv.conf` 指向 CoreDNS Service IP，`ndots:5` 會造成多次無效查詢，可調整。
- Headless Service（`clusterIP: None`）：DNS 直接回傳所有 Pod IP，不經 kube-proxy。

## 四、其他入口型態

|類型|路徑|
|---|---|
|NodePort|Client → NodeIP:NodePort → kube-proxy → Pod|
|LoadBalancer|Client → LB → NodePort → kube-proxy → Pod|
|Ingress|LB → Ingress Controller → Pod（L7 路由）|
|Gateway API|同 Ingress，資源模型分層（GatewayClass / Gateway / Route）|
|Egress（出口）|Pod → CNI → Node SNAT（MASQUERADE）→ 外網|

## 五、控制平面如何驅動資料平面

1. 建立 Service / Pod → API Server 寫入 etcd
2. EndpointSlice controller 依 Pod readiness 產生 EndpointSlice
3. kube-proxy / Ingress Controller / CoreDNS 透過 watch 取得變更並更新規則
4. 只有通過 readinessProbe 的 Pod 才會出現在 Endpoints 中

## 六、排錯順序

1. DNS：`nslookup svc.ns.svc.cluster.local`
2. Endpoints 是否為空：`kubectl get endpointslices`（label selector 不匹配、readiness 失敗）
3. kube-proxy 規則：`iptables-save | grep <svc>` 或 `ipvsadm -Ln`
4. Pod 間連通：`kubectl exec` 直連 PodIP，區分是 Service 層還是 CNI 層問題
5. NetworkPolicy 是否阻擋
6. Ingress Controller 日誌與 upstream 設定

需要的話，我可以畫成 Ingress → Service → Pod 的封包 NAT 變化圖，或展開 iptables 鏈的實際規則。