![](https://miro.medium.com/v2/resize:fit:1400/1*7s0gjRye70M4NsSsgspW3w.png)
https://medium.com/ai-blog-tw/使用者導向的kubernetes-k8s-入門課-基本架構與使用概念-2f72dc0b0fe0
### Control Plan
: 決策與組態控制
- `Kube-apiserver`:
	- Cluster 的 REST API入口，負責驗證(Authn), 授權(Authz, RBAC), Admission Control
	- 元件間溝通媒介
	- Stateless, 可水平擴展
- `kube-scheduler`
	- 監聽還沒binding Node的 Pod，透過**Filtering**（predicates，如資源、taints/tolerations、affinity）與 **Scoring**（priorities）**選 Node binding**(寫入`spec.nodeName`)
- `kube-controller-manager`: 負責執行叢集的Control-Loop, 讓系統具備Self-healing
	- Control-Loop: 持續`kube-apiserver`監視cluster共享狀態，進行調整修復
	- Node Controller
	- Replication Controller
	- Endpoint Controller
	- Namespace Controller: Namespace建立與生命週期管理
- `etcd`: 分散式 key-value store：儲存cluster狀態

### Worker Nodes
: 執行 Pod 的機器 (VM or Physical)
- `kubelet`:監聽 `apiserver`指派Node的 Podspec，透過`CRI`(Container Runtime Interface)管理container生命週期
- `kube-proxy`: 實作Service網路規則，把ClusterIP / NodePort流量給後段Pod
- `Container Rumetime`：實際執行container，需符合`CRI`
note: `Deployment`
- 管理PodReplicaSet的物件

![1*LOU2C-dTnGHumU9HAjr18w.png](https://miro.medium.com/v2/resize:fit:1400/1*LOU2C-dTnGHumU9HAjr18w.png)
**Pod Creation**
1. `kubectl apply` : `appiserver`驗證後寫入etcd
2. `Controller 建立 Pod
3. `Scheduler` 將 Pod 綁定 Node
4. Node 呼叫 `Container runtime` 啟動container
5. `kubelet` 回報 Pod Status 給 apiserver
**Pod 狀態排查**
- `Pending`: 資源不足、無符合節點、PVC未綁定，`discribe pod` 看 Events
- `ImagePullBackOff`：image name\tag錯誤、或缺 imagePullSecret
- `CrashLoopBackOff`：容器啟動後崩潰 `logs --previous`
- `Running`但不通：readinessProbe, Service selector, lable是否一致 `kubectl get endpoints <svc>` 

查看Cluster狀態
- `kubestl cluster-info`: 查看 Control plain 跟 Service 位址
- `kubectl get nodes`：查看cluster所有Node（主結點與工作節點健康狀態）
管理 Pod
- `kubectl get pods`：目前Namespace中所有Pod
- `kubectl run <pod-name> --image=<image-name>`：快速建立並指定Pod
- `kubectl describe pod <pod-name>`：查看Pod狀態、事件、錯誤訊息
- `kubestl delete pod <pod-name>`：刪除Pod
管理 Deployment
- `kubectl get deployments`：查看部署狀態
- `kubectl rollout undo deployment/nginx-deployment` 
	- 回滾到先前穩定版本
- `kubectl rollout status deployment/nginx-deployment` : 查看回滾狀態

note: [[kubectl 常用指令]]