#### Definition
：是傳輸層中 Connection-Orient, Byte-Stream, Reliable的協定，提供雙方全雙工通信
- Connection oriented：傳輸前需要先建立連接
- Byte-Stream oriented：sender 跟 receiver 的 segment byte 可以不同，但保證順序
	- `send()`、`recv()`、實際傳輸`segment` 三者完全獨立
	- ex. 一次 `send()`可能被拆成多個 `segment`
	- ex. 多次 `send()`可能合併成一個`segment`
- Reliability Delivery：保證 Sender 跟 Receiver 的 Byte Stream 完全一致（不遺失、不重複、不亂序、沒損壞）

### TCP Connection
##### Connection Creation: 3 - way handshake
![[Pasted image 20261024121026.png]]

#### Connection Termination
![[Pasted image 20261024121127.png]]


### TCP Flow Control
：用於**限制下次傳送到 Receiver 的資料大小**，Receiver 透過 Sliding window 機制 ($rwnd,$ receive window) 告知剩餘 buffer size。

![[Pasted image 20261024153148.png]]


### TCP Congestion Control(ex. TCP Tahoe)
：透過$cwnd$限制 Sender 傳送Data rate，**避免整體超過網路 link 能承受的 Data rate**$$\text{last byte send} - \text{last byte ack}≤\min\{cwnd, rwnd\}$$
![[Pasted image 20261024155508.png]]

- **Slow Start** : $cwnd = cwnd + 1$, 超過 Threshold進入 congestion avoidance (threshold $= \frac{\max cwnd}{2}$)
- **Congestion avoidance**: $cwnd = cwnd + \frac{1}{cwnd}$ ，收到三個錯誤ack 或 time out 進入 Fast retransmission
- **Fast retransmission**：重傳 MSS，$cwnd = 1$ ，threshold $= \frac{\text{當前 } cwnd}{2}$，開始 slow start


#### End to End Principle
中間層（Router）要盡可能簡單通用，只負責基本封包轉送，**複雜應用層功能應該放在系統 End point**
- 只有 Transport Layer (TCP) reliable是因為只有兩端 application可以驗證是否正確抵達，中間層不用多餘可靠性