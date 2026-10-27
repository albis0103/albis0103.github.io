
API

| Functional Requirement                                                                                                                                                                                                     | Non-Functional Requirement                                                                                      |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| $\text{SET(key, val, ttl)}$<br>: store a value against unique key                                                                                                                                                          | $\text{Low-Latency}$<br>: $P99$(percentil): $[1, 10]ms$                                                         |
| $\text{GET(key)}$<br>: retrieve a value by key                                                                                                                                                                             | $\text{Scalability}$<br>: Horizontal scaling                                                                    |
| $\text{DELETE(key)}$<br>: Remove a key value pair                                                                                                                                                                          | $\text{Availabillity}=\frac{\text{up-time}}{\text{up-time}+\text{down-time}}$ ex.$99.99\% \;:<53$ min per year  |
| $\text{TTL \& automatic eviction}$<br>**ttl-based**: eviction depends on $\text{ttl}$<br>(Lazy-expiration &active expiration)<br>**memory-presure** : when memory full, system depends eviction-policy to eviction(ex.LRU) | $\text{Reliability} = P(\{\text{system non-failure in time }\; t\})$<br>: the probability of system non-failure |
- Client SDK職責
	- 維護 partition 拓撲資訊（consistent hash ring 或 shard map）
	- Connection pooling（避免每次請求重新建立 TCP 連線）
	- 失敗重試與 fallback（read-through 到 DB
### Data storage
In node:
- LRU structure: Hash map + Doubly Linkedlist
	- get/put:$O(1)$
	- Eviction: $O(1)$
		1. Capacity reached, look $tail:\text{deque.peekLast()}$
		2. unlink $tail$ node(node C): $O(1)$
		3. $\text{map.remove('key C')}$
![[Pasted image 20260807160514.png|507]]
- 過期機制
	- Lazy expiration：讀取時檢查TTL
	- Active Expiration：背景Thread 定期掃描清除過期key
持久化:為縮短Node重啟後 Cache warm-up
- Snapshop(RDB-like)：特定時間將記憶體內所有資料Copy on write到 Disk
- Append-Only Log：先寫入Log在更新記憶體

### High Level Design

### Partition
:Consistent hashing + virtual Nodes
- 將Hash space 組成 ring
	- 每個實體node對應多個Virtual Node分散 on ring，為了：
		- 節點增減只影響相鄰Virtual Node, 不用全rehash
		- 資料分布更均勻
	- Hash(Key)沿著ring 順時針找到第一個Virtual Node的實體節點即目標
- Rebalancing
	- 新增節點：插入Virtual Node只搬鄰近資料
	- Node Failure: Node故障交給下個replica同步資料

|方案|優點|缺點|
|---|---|---|
|Range partitioning|支援範圍查詢，易於理解|容易產生 hotspot（連續 key 集中在同一 shard）|
|Hash partitioning（無 virtual node）|分佈均勻|節點增減時大量資料需重新映射|
|Consistent hashing + virtual node|增減節點影響範圍小、負載均勻|實作複雜度較高，需維護 ring metadata|

### Concurrency
**In-Node**
- Thread model
	- Single Thread + event loop
		- 避免lock contentino, 序列化執行，可透過多實例/多 shard per machine 彌補 Single Thread
		- Note: event loop
			- **等待事件**：透過多路復用機制（如 `epoll_wait`）等待Socket產生事件（如客戶端連線、發送指令、準備好接收資料）
			- **處理已就緒事件**：
			    - 把請求從 Socket 讀入記憶體緩衝區。
			    - 執行對應的 Redis 指令。
			- **寫回資料**：將執行結果寫入回覆緩衝區，並嘗試將資料寫回客戶端 Socket。
			
			- **進入下一個迴圈**（包含執行一些背景/定時任務，如 `beforeSleep` 處理）。
		- **Lock粒度**：per-key lock（衝突機率低過多會增加記憶體與 context switch ，可用 lock striping（固定數量的 lock pool，key hash 到某個 lock）折衷
		- **Lock-free 結構**：熱路徑可用 CAS（Compare-And-Swap）實作，避免鎖等待
**Between-Node(write 一致性)**
- Primary-replica架構: 先寫入Primary架構，在非同步複製到replica, only 最終一致性
- quorum洗入：保證強一致性，W 個節點 ack 後才回應 client 成功

### Availability
**Replication**
- 每個shard: 1 primary + N replica，跨機房部署
- 複寫模式
	- Async：延遲低，primary故障可能丟失
	- Semi-Sync：至少一個replica寫完回ACK
**Failover**
- Heartbeat: Coordination service(ex. etcd / Zoopkeeper)維護Node(定期發packet檢查ack)
- 投票：primary time out由coordination service選舉挑replica成primary
- Smart Client: clinet 訂閱 topology change events後, server變動control plan會push new topology to client

**Tunable Consistency**
-  N（副本數）、W（寫入需 ack 的副本數）、R（讀取需查詢的副本數）
- W + R > N 時保證讀到最新寫入（strong consistency）

**HotKey**
Problem: Consistent Hashing不保證單一Key存取頻率過高
- 統計Top-N key：用 `Count-Min Sketch`機率資料結構算key出現次數避免Map頻繁修改問題
Solution:
- L1 Cache: 
	- Client對 HotKey 縮短TTL，減少後端請求量
- Key replication：將key複製到多個節點，寫入時write - through 到 DB, replica各自TTL過期
- Request Coalescing / Single fight: 同個key多個請求只讓一個訪問後端避免Thundering(雪崩：大量Cache同時過期 or Cache壞掉)


**關鍵設計決策總結**：

| 面向           | 選擇                                              | 理由                          |
| ------------ | ----------------------------------------------- | --------------------------- |
| Partition    | Consistent hashing + virtual node               | 節點增減時遷移成本低                  |
| Concurrency  | Per-key lock / lock striping                    | 平衡吞吐與實作複雜度                  |
| Availability | Primary-replica + coordination service 選主       | 故障可自動復原，跨 AZ 容錯             |
| Consistency  | 預設 eventual，可調 quorum                           | Cache 場景優先低延遲               |
| Hot key      | L1 local cache + key replication + singleflight | 分層防禦，避免單點過載與 cache stampede |