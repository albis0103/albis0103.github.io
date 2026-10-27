
# Distributed Cache 系統設計

---

## 1. Requirements

### 1.1 Functional Requirements

- `GET(key)`、`SET(key, value, ttl)`、`DELETE(key)`
- TTL 過期機制（lazy expiration + active expiration）
- Batch 操作：`MGET`、`MSET`
- 可選：支援多種 value type（string / list / hash / sorted set），本設計以 key-value（byte array）為核心，type 層可在 storage engine 上疊加

### 1.2 Non-Functional Requirements

- **Latency**：P99 < 5ms（同機房），單機讀寫 < 1ms
- **Throughput**：假設單叢集需承載 1M QPS
- **Availability**：99.99%（年停機 < 53 分鐘）
- **Scalability**：支援水平擴充（節點數從 10 → 1000）
- **Consistency**：Cache 場景通常接受 eventual consistency，但需可調（tunable），避免 replica 落後造成髒讀影響過大
- **Durability**：非必要（cache-aside 模式下，資料遺失可從 DB reload），但需支援可選持久化以縮短冷啟動時間

### 1.3 容量估算（Back-of-envelope）

- 假設：1 億 key，平均 value 100B，平均 key 30B → 總資料量 ≈ 13GB（單份）
- 若每節點記憶體 32GB 可用容量約 24GB（保留 25% overhead）→ 至少需要 2 個 shard 承載資料，再乘上 replication factor（如 3）→ 實際部署節點數以可用性與熱點分散需求為準（通常遠大於容量需求算出的最小值）
- QPS 1M，若單節點可承載 5 萬 QPS → 至少需要 20 個 shard 做並行處理

---

## 2. Data Storage

### 2.1 單機儲存引擎

- **主結構**：Hash Table（O(1) 平均存取），bucket 內以 chaining 或 open addressing 處理碰撞
- **記憶體管理**：
    - Slab allocation（類 Memcached）：預先切分固定大小的 memory chunk，避免頻繁 malloc/free 造成的碎片化
    - 或使用 jemalloc / tcmalloc 搭配物件池
- **Eviction Policy**：
    - LRU（近似 LRU，用 sampling 方式選取候選 key，避免維護全域 linked list 的鎖競爭，Redis 即採此法）
    - LFU（適合 access pattern 長期穩定的場景）
    - TTL-based：優先驅逐已過期 key
- **過期機制**：
    - Lazy expiration：讀取時檢查 TTL 是否過期
    - Active expiration：背景 thread 定期抽樣掃描並清除過期 key，避免記憶體被大量已過期但未被存取的 key 佔用

### 2.2 持久化（可選）

- **Snapshot（RDB-like）**：定期 fork 子行程做 point-in-time dump，利用 copy-on-write 降低對主行程的影響
- **Append-Only Log（AOF-like）**：寫入操作先寫 log 再更新記憶體，重啟時 replay log 復原狀態；可搭配定期 compaction 縮小 log 體積
- 持久化目的是縮短節點重啟後的 cache warm-up 時間，非強一致性保證

### 2.3 序列化

- Value 以 byte array 儲存，序列化格式（Protobuf / MessagePack）由 client 端決定，cache node 對內容不感知（agnostic），僅處理 byte 層級的儲存與傳輸

---

## 3. API Design

```
GET    /cache/{key}                → 200 {value, ttl} | 404
PUT    /cache/{key}                body: {value, ttl_seconds} → 200 | 201
DELETE /cache/{key}                → 200 | 404
POST   /cache/batch/get            body: {keys: [...]}   → {key: value, ...}
POST   /cache/batch/set            body: {entries: [{key, value, ttl}]}
GET    /cache/{key}/ttl            → {ttl_remaining}
```

- 若走 RPC（gRPC）而非 REST，可降低 serialization overhead 與連線建立成本，實務上高效能 cache client 多採自訂二進位協議（如 Redis RESP、Memcached binary protocol）
- **Client SDK 職責**：
    - 維護 partition 拓撲資訊（consistent hash ring 或 shard map）
    - Connection pooling（避免每次請求重新建立 TCP 連線）
    - 失敗重試與 fallback（read-through 到 DB）

---

## 4. High Level Design

### 4.1 Partition（分片）

**方案：Consistent Hashing + Virtual Nodes**

- 將 hash space（如 0 ~ 2^32-1）組成一個環（ring）
- 每個實體節點對應多個 virtual node（如 150 個），分散掛載在環上，目的：
    1. 節點增減時，資料遷移只影響相鄰的 virtual node，不需全量 rehash
    2. Virtual node 數量夠多時，資料分佈更均勻（避免單一節點負載過高）
- Key 經過 hash function（如 MurmurHash / xxHash）映射到環上一點，沿順時針方向找到第一個 virtual node 所屬的實體節點即為目標

**Rebalancing**：

- 新增節點：在環上插入其 virtual node，僅需搬移相鄰區間的資料
- 節點下線：其負責的 key range 交由環上下一個節點接管，該節點須從 replica 或原節點（若可達）同步資料

**替代方案比較**：

|方案|優點|缺點|
|---|---|---|
|Range partitioning|支援範圍查詢，易於理解|容易產生 hotspot（連續 key 集中在同一 shard）|
|Hash partitioning（無 virtual node）|分佈均勻|節點增減時大量資料需重新映射|
|Consistent hashing + virtual node|增減節點影響範圍小、負載均勻|實作複雜度較高，需維護 ring metadata|

### 4.2 Concurrency（並行控制）

**單節點內部**：

- **執行緒模型**：
    - 單執行緒 + event loop（如 Redis）：避免鎖競爭，所有指令序列化執行，簡化一致性推理，但無法利用多核心（可透過多實例/多 shard per machine 彌補）
    - 多執行緒 + 分段鎖（如 Memcached）：Hash table 依 bucket 分段，每段獨立加鎖（lock striping），提升並行度但增加實作複雜度
- **鎖粒度**：per-key lock（細粒度，衝突機率低）優於 global lock，但過多鎖物件會增加記憶體與 context switch 開銷，可用 lock striping（固定數量的 lock pool，key hash 到某個 lock）折衷
- **Lock-free 結構**：熱路徑（如計數器型操作 INCR）可用 CAS（Compare-And-Swap）實作，避免鎖等待

**跨節點（write 一致性）**：

- 若採 primary-replica 架構，寫入先落 primary，再非同步（或半同步）複製到 replica，期間讀 replica 可能讀到舊值（stale read）
- 需要強一致性時，可用 quorum 寫入（W 個節點 ack 後才回應 client 成功）

### 4.3 Availability（可用性）

**複寫（Replication）**：

- 每個 shard 配置 1 個 primary + N 個 replica（如 N=2），跨 AZ 部署避免單一機房故障造成資料遺失
- 複寫模式：
    - Async：延遲低，但 primary 故障時可能遺失尚未同步的寫入
    - Semi-sync：至少 1 個 replica 確認收到才回應 client，平衡延遲與資料安全

**Failover**：

- **健康檢查**：Heartbeat 機制，coordination service（如 etcd / ZooKeeper）維護節點存活狀態
- **故障偵測與選主**：primary 失聯超過 threshold 時，由 coordination service 觸發選舉，從 replica 中挑選資料最新者升級為新 primary（類似 Raft leader election）
- **Client 端**：訂閱 topology 變更事件，即時更新路由表，避免持續打向已下線節點

**Tunable Consistency（Dynamo-style，可選）**：

- 定義 N（副本數）、W（寫入需 ack 的副本數）、R（讀取需查詢的副本數）
- W + R > N 時可保證讀到最新寫入（strong consistency）；降低 W/R 可換取更低延遲，犧牲一致性強度

### 4.4 Hot Key（熱點問題）

**問題本質**：Consistent hashing 只保證「不同 key」分佈均勻，無法解決「單一 key 存取頻率過高」導致該 key 所在節點/分片負載遠高於其他節點（如秒殺場景下某商品 key 被大量併發存取）。

**偵測**：

- 節點端即時統計 top-N 存取頻率（可用 Count-Min Sketch 這類機率性資料結構，以 O(1) 空間近似估算高頻 key，避免對每個 key 都維護精確計數器造成額外開銷）

**解法**：

1. **本地快取（多級快取 / L1 cache）**：Client 端或 proxy 層對偵測到的熱點 key 做短 TTL（如數百 ms）的本地快取，減少打到後端節點的請求量
2. **熱點打散（key replication）**：將熱點 key 複製到多個節點（如 `hotkey#0` ~ `hotkey#9`），client 端隨機選一個副本讀取，寫入時需同步更新所有副本或改採 write-through 到 DB + 各副本各自 TTL 過期
3. **Request Coalescing / Singleflight**：同一時間對同一 key 的多個併發請求，只讓其中一個真正打到後端（如 DB 或原始節點），其餘請求等待並共用結果，避免 cache miss 瞬間造成 thundering herd（快取雪崩到 DB）
4. **動態分片**：偵測到熱點後，將該 key 所在的 virtual node 範圍進一步切細，並遷移到專屬節點，隔離對其他 key 的影響

---

## 5. Complete Architecture

```
                         ┌─────────────────────┐
                         │   Coordination       │
                         │  Service (etcd/ZK)   │
                         │  - Ring / Shard Map   │
                         │  - Node Health        │
                         └───────────┬───────────┘
                                     │ watch topology
                 ┌───────────────────┼───────────────────┐
                 │                   │                   │
          ┌──────▼──────┐    ┌──────▼──────┐     ┌──────▼──────┐
          │  Client SDK  │    │  Client SDK  │     │  Client SDK  │
          │ (routing +   │    │ (routing +   │     │ (routing +   │
          │  pooling +   │    │  pooling +   │     │  pooling +   │
          │  L1 hotkey   │    │  L1 hotkey   │     │  L1 hotkey   │
          │  local cache)│    │  local cache)│     │  local cache)│
          └──────┬───────┘    └──────┬───────┘     └──────┬───────┘
                 │                   │                    │
                 └─────────────┬─────┴────────────────────┘
                                │  (可選 Proxy 層，如 Twemproxy/Codis，
                                │   集中處理路由與連線多工，降低 client 複雜度)
                 ┌──────────────┼──────────────┐
                 │              │              │
          ┌──────▼─────┐ ┌──────▼─────┐ ┌──────▼─────┐
          │  Shard 0    │ │  Shard 1    │ │  Shard N    │
          │ ┌─────────┐ │ │ ┌─────────┐ │ │ ┌─────────┐ │
          │ │ Primary  │ │ │ │ Primary  │ │ │ │ Primary  │ │
          │ └────┬────┘ │ │ └────┬────┘ │ │ └────┬────┘ │
          │      │repl   │ │      │repl   │ │      │repl   │
          │ ┌────▼────┐ │ │ ┌────▼────┐ │ │ ┌────▼────┐ │
          │ │Replica×N│ │ │ │Replica×N│ │ │ │Replica×N│ │
          │ └─────────┘ │ │ └─────────┘ │ │ └─────────┘ │
          └─────────────┘ └─────────────┘ └─────────────┘
                 │ miss           │ miss           │ miss
                 └────────────────┼────────────────┘
                                  ▼
                         ┌─────────────────┐
                         │  Backing Store   │
                         │  (Primary DB)    │
                         └─────────────────┘

          ┌──────────────────────────────────────────┐
          │ Monitoring / Metrics（延遲、QPS、eviction │
          │ rate、hot key 偵測、replica lag）          │
          └──────────────────────────────────────────┘
```

**單節點內部結構**：

```
Network Layer (event loop / epoll)
    → Command Parser
    → Concurrency Control (per-key lock / lock-free CAS)
    → Storage Engine (hash table + slab allocator)
    → Eviction Manager (LRU/LFU sampling)
    → Expiration Manager (lazy + active)
    → Replication Manager (async/semi-sync stream to replica)
    → Persistence Manager (optional snapshot/AOF)
```

**關鍵設計決策總結**：

| 面向           | 選擇                                              | 理由                          |
| ------------ | ----------------------------------------------- | --------------------------- |
| Partition    | Consistent hashing + virtual node               | 節點增減時遷移成本低                  |
| Concurrency  | Per-key lock / lock striping                    | 平衡吞吐與實作複雜度                  |
| Availability | Primary-replica + coordination service 選主       | 故障可自動復原，跨 AZ 容錯             |
| Consistency  | 預設 eventual，可調 quorum                           | Cache 場景優先低延遲               |
| Hot key      | L1 local cache + key replication + singleflight | 分層防禦，避免單點過載與 cache stampede |