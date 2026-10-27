## Content Delivery Network
: 由分散的邊緣伺服器(PoP, Point of Presence)組成，用於加速內容傳輸與降低Server負擔

#### 關鍵步驟
- Cache : CDN 儲存網站或 App的內容
- 就近拜訪
	1. Client → Resolver → 權威 DNS
	2. 權威 DNS 回傳：203.0.113.10（anycast IP，全球固定） 
	3. Client → 203.0.113.10 ─ BGP 就近 → 最近的 PoP （DNS 不在這條路徑上）
- Load Balence：CDN可以智慧流量分發
- 安全性：可以防DDoS

#### Netflix
![[Pasted image 20261016225728.png]]
- Open Connect：Netflix自建的CDN
	- OCA at Peering Point
		- 集中部署在少數關鍵節點(IXP, Internet Exchange Point)
		- **ISP 透過 Peering** 互連：`Client → ISP 網路 → [peering] → Netflix OCA（IXP）`[[ISP (Internet Service Provider)]]
	- OCA on ISP Premise (Embedding OCA)
		- OCA**安裝在ISP機房內**
		- 更接近Clinet
- note : Streaming
	- 邊接收邊處理的傳輸方式，把內容切成小片段，收到足夠初始資料即可開始播放。
		- ex. MPEG-DASH