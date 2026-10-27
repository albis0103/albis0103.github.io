
## 1. 快取穿透（Cache Penetration）

**定義**：查詢的資料在快取與資料庫中都**不存在**，因此每次請求都 miss 快取、並落到 DB，且 DB 查無結果、無法回填快取。

**成因**：惡意請求（如 id = -1、隨機 UUID）或業務 bug。

**對策**：

- **快取空值**：查無資料時寫入 `null` 並設短 TTL（如 30–60 秒）
- **Bloom Filter**：確定「不存在」的 key Cache直接拒絕（有 false positive、無 false negative）
- 請求層參數校驗、IP 限流

## 2. 快取擊穿（Cache Breakdown / Hot Key Expiration）

**定義**：**單一 HotKey** 在過期瞬間，大量HotKey並發請求同時 miss，全部打到 DB 重建快取。

**成因**：高 QPS 的熱點 key TTL 到期。

**對策**：

- **互斥鎖（mutex）**：只允許一個請求查 DB 並重建快取（如 `SET lock NX EX 10000`），其餘等待或重試
- **邏輯過期**：value 內存放過期時間，過期後由單一執行緒非同步重建，其他請求先回傳舊值
- 熱點 key 不設 TTL，由背景任務主動更新

## 3. 快取雪崩（Cache Avalanche）

**定義**：**大量 key 同時過期**，或**快取服務整體不可用**（如 Redis 當機），導致流量全部湧向 DB，造成 DB 過載甚至宕機。

**對策**：

- **TTL 加隨機抖動**：`TTL = base + random(0, Δ)`，避免集中過期
- **高可用架構**：Redis Sentinel / Cluster、主從複製
- **多級快取**：本地快取（Caffeine）+ Redis
- **限流、熔斷、降級**：保護 DB（如 Sentinel、Hystrix）
- 快取預熱

## 比較

| |穿透|擊穿|雪崩|
|---|---|---|---|
|影響範圍|不存在的 key|單一熱點 key|大量 key／整個快取層|
|資料是否存在於 DB|否|是|是|
|核心問題|無效查詢無法被快取吸收|單點過期 + 高並發|同時失效或服務故障|
|主要解法|Bloom Filter、空值快取|互斥鎖、邏輯過期|TTL 隨機化、高可用、限流降級|
