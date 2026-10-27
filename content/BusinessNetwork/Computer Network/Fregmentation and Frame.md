#### Def
- 將下層資料大的PDU(Protocol Data unit)切割成符合上層可階收的MTU (Maximum Transmission Unit)

#### Reason
- **MTU**(Maximum Transmission Unit)
	- link有最大傳輸單元限制
	- ex. Ethernet MTU : 1500 byte
- **Error Control**
	- bit錯誤率 $p$ ，封包長度 $L$ ，整個封包正確機率 $(1-p)^L$，即 $L$ 越大錯誤機率高
	- 出錯時透過 Error Control (ex. Selective)，不用重傳完整封包
	- Ethernet FCS：只偵測不更正，error 丟棄，不要求重傳
- MultiPlexing
	- 資料太長會造成 Head - of - line - blocking，分段後不同來源可以交錯傳送
	- ex. contention - Based: CSMA-CD [[CSMA-CD]]


#### not all Frame need header, payload, trailer
- Control Frame : payload可以為空
	- HDLC的 S-frame （傳控制訊號）
	- 802.11 的 RTS, CTS, ACK
- 不用Trailer
	- ATM cell：header + payload

	  