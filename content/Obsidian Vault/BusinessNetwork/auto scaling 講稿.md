### **請求 到 目標服務的完整過程**，Google cloud 怎麼處理
- **主要流程：四階段**
- **左邊有畫圖**讓大家清晰理解

- Front end : **反向代理**，會終止TLS
	- Ingress check: IP
		- Cloud balancer
		- **K8S ingress controller**
	- IAM: HTTP
	- Traffic split
		- **開發者部署新版本 Revision**，設定 Traffic split 把流量分配到正確的版本


- Request Routing layer: 選 instance (**運算資源**)
	- slot: **開發者**部署 n concurrency
		- 有：建 sandbox (gvisor and gen2差別 system call 顆粒度)，載入image 到 container讓他接收請求
		- 無：真正 **autoscaling**
			- Buffering queue
			- Autoscaler計算期望instance數
			- Request Routing layer新增instance
- 出站流量
	- direct VPC egress：**分派目標資源的Subnet IP** 給 container
	- VPC connector：建立 Bridge VM


###  Claude run是 request-based，用knative架構說明怎麼實現 cold start

- Activator：
	- 接收使用者請求，**追蹤pod (instance)容量**去路由正確pod
	- 沒有instance會buffering 請求，實現cold start
- Queue proxy
	- 跟**服務container在同一個pod的sidecar**
	- 請求超出 concurrency會buffering 請求
- autoscaler
	- 是KPA的決策元件
	- 收集 activator跟 queue proxy的指標計算期望副本數
- 

### Autoscaling 最大問題 Thrashing 問題
- **右下角圖與 OS Thrashing問題相同**，但情境不同
- autoscaling thrashing：偵測指標不斷在concurrency附近抖動，一直不斷增減pod 數導致服務不穩定
- 所以這裡的**最關鍵的是偵測指標**
	- HPA與KPA指標差異
	- 
- 
