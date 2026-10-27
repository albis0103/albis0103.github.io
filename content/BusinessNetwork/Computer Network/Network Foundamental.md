
#### Multiplexing
: 多個裝置連結在一起透過同一媒介傳輸
- TDM ex. T1
- FDM ex. CATV
- SM (Statistical Multiplexing) : 依實際需求分配頻寬 ex. ATM
- CDM：透過編碼正交性多工 ex. CDMA


### 網路中資料交換

#### Packet Switching
定義：
- 將資料切割成封包，透過路由器 Stored and Forward方式將資料從傳送端送到接收端
特性
- 不用事先建實體路徑
- Statistical Multiplexing：依需求提供頻寬，不會獨佔link
- 不保證資料順序： queue滿了會發生queueing delay 跟 packet loss
- 可能 Congestion：不會預留資源


#### Virtual Circuit
定義
- 來源與目的先建立虛擬路徑 (Virtual Path)，每個交換器維護 VP狀態，封包依 VCI (Virutal Channel Indentifer)轉送
流程
1. Set-up：傳送端告知接收端 IP，路徑上每個 Switch 配置 VC# (VCI)，在 forwarding table 紀錄（in - port, in - VC#）$\rightarrow$ (out - port, out VC#)
	- ex. ATM : (VPI, VCI)
2. Data Transform：Header包含 VC#，資料傳送每 hop 只更新 VC#
3. Term down：釋放 Virtual Path

### Delay
- Propagation Delay：**訊號在媒介上傳輸**時間$$\frac{\text{distance}}{\text{propagation speed}}$$
- Transmission Delay：**訊號進入媒介**時間$$\frac{\text{Data}}{\text{Media Data Rate}}$$
- Queueing Delay：Data waiting time
- Process Delay： Data process time
- Roud-trip Delay：訊號(ex. data)從來源到目的地，目的地再回傳訊號 (ex.ack)的時間

### Transmission metric
- **利用率 Utilization**$$\frac{\text{Transmission Speed}}{\text{Roud-trip Delay}}$$
- **Throughput**：頻道中資料實際成功傳送的速率
- **Rated speed** (Media Data rate)：link傳輸的理論最大速率
	- Shannon Throrem
		- 訊號雜訊比：用分貝不直接用 $S/N$ 值
$$dB = 10\;\log_{10}S/N$$
$$\text{Max Data rate}=B\log_2(1+S/N)$$


### Baseband v.s. Broadband
#### Baseband
- 將原始訊號（未調變成載波）直接送上媒介
- 只有單一通道使用媒介整個可用頻寬
#### Broadband
- 將訊號調變成載波訊號
- 媒介頻寬切分多個 channel，各channel 傳送不同訊號



#### Baseband v.s. Broadband Transmission

| 項目               | Baseband                                        | Broadband                                          |
| ---------------- | ----------------------------------------------- | -------------------------------------------------- |
| **訊號**           | 數位基頻訊號（如編碼後的方波）                                 | 類比載波調變訊號                                           |
| 頻譜               | 佔滿整個媒介頻寬                                        | 切成多個頻道，各佔一段                                        |
| **Multiplexing** | TDM, CSMA/CD                                    | FDM                                                |
| 通道數              | 單一通道                                            | 多通道同時傳輸                                            |
| **傳輸距離**         | 較短（訊號衰減、失真無放大器補償，需 repeater）                    | 較長（可用 amplifier 放大）                                |
| 代表               | Ethernet（10BASE-T、1000BASE-T，"BASE" 即 baseband） | 有線電視 CATV、DOCSIS 纜線網路、早期 10BROAD36、ADSL（頻率分割語音與資料） |

#### Baseband v.s. Broadband Channel


| 項目       | Baseband                         | Broadband                                        |
| -------- | -------------------------------- | ------------------------------------------------ |
| **頻率**   | 低通( Loss-pass)<br>頻率範圍：$0\sim B$ | 帶通 (Band-pass)<br>範圍：$f_1 \sim f_2$ 頻寬：$f_2-f_1$ |
| **容量**   | $C=B\log_2(1+S/N)$               | $\sum C_i \quad C_i = B_i \log (1 + S/N_i)$      |
| 通道數      | 單一通道                             | 多通道同時傳輸                                          |
| **頻道間隔** | 無                                | 需要 Guard band 避免頻道干擾                             |
| 適用訊號     | 低通（數位基頻訊號）                       | 帶通（調變後載波訊號）                                      |
|          |                                  |                                                  |

### Telecommunication v.s. Data Communication
#### Telecommunication
- 定義：以電、電磁波、光訊號，在兩端傳遞資訊與基礎設施總稱。
- 範圍：傳輸媒介（銅纜、光纖、無線）、交換（電路交換、封包交換）、多工（FDM、TDM、WDM）、調變、公共網路（PSTN、行動網路 4G/5G）
#### Data Communication
- 定義：兩個裝置間透過媒介交換**數位資料**，由協定規範格式與流程
- 五大組成：sender, receiver, message, transmission medium, protocol
- 範圍：編碼與調變、錯誤偵測與更正、流量控制、Data-Link層、網路協定堆疊
- 關注點：資料如何可靠、有序地在端點傳送

**關係**

| 面向   | Data Communication   | Telecommunication    |
| ---- | -------------------- | -------------------- |
| 傳輸內容 | 數位資料                 | 語音、影像、資料等各類資訊        |
| 層次側重 | 協定、資料連結層至應用層         | 實體層、傳輸網路、營運基礎設施      |
| 典型技術 | Ethernet、TCP/IP、HDLC | PSTN、SDH/SONET、蜂巢式網路 |

#### Internet
- 基於兩者實作 （L1 : Telecommunication, L2 ~ L5：Data Communication）
- 是 Best - effort Delivery System

#### Internet v.s. Single Network

|     | Single Network                           | Internet            |
| --- | ---------------------------------------- | ------------------- |
| 定義  | 單一同質網路技術(ex. Ethernet LAN)<br>組成，由單一實體管理 | 多個異質網路，有Router互連    |
| 定址  | 內部位址MAC (EUI-64)                         | 全域 IP (IPv4 / IPv6) |
| 設備  | Bridge / Switch                          | Router              |
| 路由  | Spanning Tree, Learning Table            | iBGP (OSPF) + eBGP  |
| 資料  | Frame                                    | Packet              |
##### Internet 為了解決不同 single network 互連三大問題
- 標準不同，看不懂 Frame
- DLL位置可能重複，雖然Ethernet MAC保證全球唯一，但還有其他標準其DLL位置 bit 不同
##### Solution
- Router： 在 Network 1 是目的 Host, 在 Network 2 是 來源 Host
- IP address：全球唯一
- IP packet：每經過一個 Single Network 把舊的 Frame 拆掉，Encapsulation 進 新網路的 Frame