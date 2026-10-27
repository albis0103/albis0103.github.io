#### Threshing / Flatting problem
：HPA在監控指標(HPA controller每個`sync`週期監控CPU , Memory Utilization) 頻繁波動導致 Pod replica 不斷增減 （ex. 異常流量、JVM啟動..etc)造成服務不穩定


![[Pasted image 20261014162038.png]]

#### K8S HPA solution : Stabilization window
- 擴充條件需 window 範圍內全數符合
-  Window length : $W$ , time : $t$ 
- window內推薦集合: $R = \{r(\gamma)| t-W≤\gamma≤t\}$ 
- **scale Down**：window 內全部推薦值 $R$ 低於 當前 replica, 縮減到 $\max (R)$ 
- scale up ：window 內全部推薦值 $R$ 高於 當前 replica, 擴增到 $\min (R)$ 


#### Knative KPA solution : 
- Stable window
	- 參數：`autoscaling.knative.dev/window`，預設 60s（可設 6s–3600s）。
	- 對過去 60s 的 concurrency 取平均來決定 desired pods，作用是平滑，避免短暫波動造成 pod 數震盪。
	- stable mode 下可 scale up，也可 scale down。scale down 要等整個 stable window 的平均都降下來才會發生，因此縮容是延遲的。

- Panic window
	- 參數：`panic-window-percentage`，預設 10.0，即 panic window = stable window × 10% = 6s。
	- 對最近 6s 的 concurrency 取平均，用來偵測突發流量，反應比 stable window 快。