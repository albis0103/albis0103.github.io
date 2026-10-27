
|     | QoS(Quality of Service)                                                                                                                       | SLA(Service Level Agreement)                                                                                   |
| --- | --------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| Def | 用於控制資源、分配資源的**技術機制**，讓服務達到所需水準                                                                                                                | 供應商與客戶間的**契約文件**，規定可量化服務承諾（保證不低於 worst case）                                                                   |
| 內容  | Speed: 傳輸速度 (bps)<br>Availability：網路可供使用時間 （百分比）<br>Error Rate：$\frac{\text{loss 的 bits}}{packet bits}$ <br>Latency：封包傳遞延遲<br>Jitter：封包延遲的變異 | availability ≥ 99.9 % / 月<br>Error - rate ≤ 0.001 %<br>Latency / Jitter:<br>P95 < 300 ms, <br>P99 < 800 ms<br> |
| p   | 效能下降                                                                                                                                          | 違約                                                                                                             |
#### Latency v.s. Jitter
![[Pasted image 20261024233158.png]]
- Latency: 封包延遲時間
- Jitter
	- 延遲的變異 $\frac{\text{Latency 平均變異}}{\text{平均 latency}}$ 
	- 將低Jitter需要很大成本，因為需要 over - provisioning（配置資源容量遠大於預期需求）
	- VoIP 特別敏感，因為 Jitter會讓封包間隔不均勻，播放斷斷續續

### Momentary Traffic peaks
在極短時間內，請求到達率超過服務速率($\lambda(t) > \mu$)，但長期來看，平均到達率 $\lambda < \mu$ ![[Pasted image 20261025010148.png]]
#### Solution
- Overprovisioning：提供比預期需求更高的容量（network capacity）
	- 成本高、浪費
- Priority：高傳輸量優先處理 ex. 聲音 > email
	- 需要更多人力設定
- Reservation：RSVP 預留容量
	- 浪費資源，預留的可能沒用

### Congestion Prevention
在資源耗盡前（Momentary Traffic peak 前）對 Congestion 介入處理
- Network Layer: QoS
- Transport Layer: TCP Congestion Control, ECN
- Application Layer: Circuit Breaker, Autoscaling