#### 定義
- 將 Control Plane 與 Data Plane 分離，透過可程式化的軟體控制器操作控制邏輯。
- 控制器負責路由與決策，透過**標準化介面(OpenFlow)**下發
- Switch 只負責依 **flow table** 轉發封包

#### 架構

|                      | 內容                                     | 介面                        |
| -------------------- | -------------------------------------- | ------------------------- |
| Application Layer    | 負載平衡、防火牆、流量工程、網路監控等應用                  | Northbound API（REST、gRPC） |
| Control Layer        | SDN Controller：維護拓樸、計算路徑、管理 flow table | Southbound API（OpenFlow）  |
| Infrastructure Layer | 支援 SDN 的交換器／路由器（資料平面）                  | —                         |

#### OpenFlow 
：標準化南向協定，定義 Controller 與 Switch溝通方式
- **Flow Table**：路由規則，由多筆 flow entry 組成，每筆包含：
	- **Match Fields**：in_port、eth_src/dst、eth_type、VLAN、IP src/dst、IP proto、L4 port 等
	- **Priority**：多筆比對成功時取最高優先權
	- **Counters**：封包／位元組計數
	- **Instructions / Actions**：output、drop、set-field、goto-table 等
	- **Timeouts**：idle_timeout、hard_timeout
- 流程
	1. 封包抵達交換器，查 flow table。
	2. 命中 → 依 action 轉發，更新 counters。
	3. 未命中 → 依 table-miss 送 Packet-In 給控制器。
	4. 控制器計算路徑，回送 Flow-Mod 安裝 entry，並以 Packet-Out 轉發該封包。
	5. 後續同 flow 封包直接在資料平面命中。

### 常見 Controller
|控制器|語言|特性|
|---|---|---|
|ONOS|Java|分散式、電信級、支援 intent framework|
|OpenDaylight|Java|Linux Foundation 專案，模組化（OSGi）|
|Ryu|Python|輕量，適合研究與教學|
|Floodlight|Java|早期廣泛使用|
|POX / Faucet|Python|教學／production（Faucet 以 OpenFlow 管理校園與 IXP 網路）|

