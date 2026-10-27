#### Hierarchical Ethernet LAN
- 點到點之間只有單一路徑
- Switch table 中 同個 `MAC address` 只會出現一次

- Workgroup Switch ：透過 Access link(ex. UTP) 連接主機
- Core Switch：Trunk link(光纖為主)，連接 Switch。需要處理許多 Session，處理能力要強

![[Pasted image 20261025000730.png]]

#### Frame 轉發流程：
1. **Learning**：檢查 Frame `source MAC`與 `source port` 
	- entry不存在： new `MAC`, `Port`
	- entry 存在：
		- port 相同：不新增
		- port 不同：直接覆寫（視為主機移動）
2. Switching Table：用 `Destination MAC`查 `Switching TABLE` 的 Port
3. 檢查 `Destination Port`與`Source Port`
	- 若相同，**Filtering** 過濾
	- 若不同，**Forwarding** 轉發
	- 若 `Destination Port` 未知，**Flooding** 轉算給 VLAN內所有 Port
##### Flooding會造成 loop，怎麼解決
- 透過[[STP and RSTP]]建構 spanning tree

#### Switch 轉發方式
- Store and Forward：收完完整 Frame 才轉發
- Cut - Through：讀到 Header 中 Destination MAC就開始轉發
- Fragment - Free：讀滿 64 bytes轉發


#### Router轉發流程差異
1. 取 Destination IP 位置
2. Routing table 中 會**有多個匹配的 Entry**，**選 Longest Prefix Matching**
	- `10.1.2.0/24`（選這個）
	- `10.1.0.0/16`

##### Note
- L2 : Header , Trailer 每個網路都會換，L3: IP位置不變，描述整條 route