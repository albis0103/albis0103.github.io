: Google 2018 發起的專案，跑在 K8S上的 serverless 平台層
- Knative Serving：一個Container + Configuration 變成可 Auto-scaling, 縮到 0 的 HTTP服務
- Knative Eventing：事件驅動的 broker , CloudEvents
#### Knative Serving
![[Pasted image 20261013010315.png]]
- Activator : when **Revision $\rightarrow 0$**，接受並Buffer請求，通知 autoscaler 擴充，Pod `ready` 後轉發request 
- Control Plane
	- Autoscaler:是 Knative 的 Deployment![](https://miro.medium.com/v2/resize:fit:1400/1*RY1KILK0XDzoO9cZNhIewg.png)
		- 收集metrics 計算目標replica數， KPA(Knative Pod Autoscaler)決策核心
$$

\text{desiredReplicas} = \left\lceil \frac{\text{observedConcurrency}}{\text{target} \times \text{targetUtilization}} \right\rceil

$$

- **observedConcurrency**：時間窗口內的平均 Concurrency 請求數（所有 Pod 加總）
- **target**：每個 Pod 的目標 Concurrency 數，由 `autoscaling.knative.dev/target`（或 `containerConcurrency` 的 soft limit）設定，預設 100。
- **targetUtilization**：`container-concurrency-target-percentage`，預設 70%，因此實際有效目標是 70。

- Data plane
	- **queue-proxy (sidecar)**：與用戶 Service 容器在同個Pod
		- 當請求超出 Concurrency ，會把多餘請求Buffer在 queue，當前Service結束再處理剩下request
		- 回報 concurrency / RPS 給 autoscaler




### Knative Eventing

提供**鬆耦合、事件驅動**架構，事件格式採用 **CloudEvents** 標準。

### 3.1 核心概念

|元件|職責|
|---|---|
|**Source**|事件產生者（`ApiServerSource`、`PingSource`、`KafkaSource`、`ContainerSource`、`GitHubSource` 等），將外部事件轉為 CloudEvents|
|**Sink**|事件接收者，可為 Knative Service、K8s Service、Channel、Broker 或 URI|
|**Broker**|事件匯集點，接收事件並依 Trigger 分發|
|**Trigger**|訂閱 Broker 的過濾規則（依 CloudEvents attributes 如 `type`、`source`），將符合的事件送往 subscriber|
|**Channel / Subscription**|較低階的 pub/sub 抽象；Channel 為事件傳輸層，Subscription 連結 Channel 與 subscriber|
|**Sequence / Parallel**|事件處理的串接與平行分支|
|**EventType**|事件目錄，描述 Broker 中可用的事件類型|

### 3.2 常見拓樸

**Broker/Trigger 模型（推薦）：**

```
Source ──► Broker ──Trigger(type=A)──► Service A
                 └─Trigger(type=B)──► Service B
```

**Channel/Subscription 模型：**

```
Source ──► Channel ──Subscription──► Service
                  └─Subscription──► Service
```

### 3.3 Channel/Broker 實作（可插拔）

- `InMemoryChannel`：僅供開發，無持久化
- **Kafka**（`KafkaChannel` / Kafka Broker）：生產常用，具持久性
- `NATS Streaming`、`RabbitMQ` 等

### 3.4 傳遞語意

- 預設為 **at-least-once**，subscriber 需設計為冪等
- 支援 **retry、backoff、dead letter sink（DLS）** 設定
- Subscriber 回傳 2xx 視為成功；回傳的 CloudEvent 可被注入回 Broker，形成事件鏈

### 3.5 與 Serving 結合

Sink 指向 Knative Service 時，事件到達會透過 Serving 的 activator/autoscaler 機制從 0 擴容，達成**事件驅動 + scale-to-zero**。

---

## 4. 網路層（Networking）

Knative 對 ingress 實作抽象化，透過 `Ingress` CRD（`networking.internal.knative.dev`）與外掛式 controller 對接：

|實作|特性|
|---|---|
|**Kourier**|輕量、基於 Envoy，專為 Knative 設計，資源占用低|
|**Istio**|功能完整（mTLS、細粒度流量策略），資源開銷較大|
|**Contour**|基於 Envoy|
|**Gateway API**|標準化 K8s Gateway API 實作|

另有 **DomainMapping**（自訂網域）、**cert-manager** 整合（自動 TLS）、**external-dns** 搭配。

