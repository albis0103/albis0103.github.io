#### Implement 1
：find ICMP type 0/3/8
1. find Network Interface
```bash
route -n get default 
```
![[Pasted image 20261013123445.png]]or
```bash
route -n get default | grep interface
```
![[Pasted image 20261013123553.png]]

2. `ping 8.8.8.8` check wireshark

![[Pasted image 20261013111619.png]]

![[Pasted image 20261013111706.png]]

- Type 8  Echo Request ：由發送端 Client 發出，回覆請求
- Type 0 Echo Reply：由 接收端 Server發出，回覆應答

![[Pasted image 20261013111345.png]]
192.0.2.1 屬於 TEST-NET-1，熱點的上游直接丟棄封包，所以沒有 Type 3，只有timeout。

補上 type 3 : 對閘道的關閉 UDP port 送封包，閘道會回 Destination Unreachable

![[Pasted image 20261013111952.png]]


#### Implement 2
: find ftp USER PASSWORD

1. used following TCP Stream
![[Pasted image 20261013122311.png]]
2. used Display Filter
![[Pasted image 20261013122551.png]]

**替代加密方案

1. **SFTP**（SSH File Transfer Protocol）：跑在 SSH（TCP 22）之上，控制與資料通道共用單一加密連線，帳密與檔案內容都被加密。
	1. 過濾器 `ssh`。可看到 SSH 版本交換與 Key Exchange Init，之後的封包全是加密的 Encrypted packet。
2. **FTPS**（FTP over TLS/SSL）：在 FTP 上加 TLS，可用 explicit（`AUTH TLS`，port 21）或 implicit（port 990）。控制通道與資料通道都能加密，並以憑證驗證伺服器。
	1. 過濾器 `ftp || tls`。會看到明文的 `AUTH TLS` 與 `234` 回應，之後變成 `TLSv1.x` 的 Client Hello / Application Data，內容無法還原。

#### Implement 3
: check wireshark follow TCP Stream

![[Pasted image 20261013123924.png]]

- Telnet: TCP port 23: 提供遠端 Server 間 連線的虛擬終端機，讓使用者透過Internet 登入遠端 Server
- 其特性：
	- 已明文傳輸
	- 使用者輸入字元需先轉成虛擬終端機格式，虛擬終端再轉成 Server 所需格式
	- 允許使用者與Server進行 Option協商機制

##### Telnet 呈現與FTP不同原因 逐字元傳輸
：Telnet 互動模式，Client 按一個鍵就送出一個TCP ，由 Server echo字元呈現在虛擬終端機
- 虛擬終端換行：收到 `\r\n`
- wireshark Follow TCP Stream：把每個TCP Segment資料接起來
```
Client→Server: a
Server→Client: a
Client→Server: d
Server→Client: d
...
```