
## 1. Timeout

**定義**：對一次操作的等待時間設定上界，超過即主動放棄並回報失敗。作用是把「無上界的延遲」轉換成「明確的失敗」。

**為什麼必要**：呼叫端等待期間會持續佔用 thread、connection、記憶體。下游變慢但沒有 timeout 時，呼叫端資源會被耗盡，故障向上游擴散（cascading failure）。

**類型**

|類型|計時範圍|
|---|---|
|Connection timeout|TCP/TLS 連線建立完成前|
|Read (socket) timeout|連續兩次收到資料之間的間隔，不是整個回應的總時間|
|Write timeout|送出請求資料的時間|
|Request timeout / deadline|單次請求從發出到完整收到回應的總時間|
|Pool acquisition timeout|從 connection pool 取得連線的等待時間|

只設 read timeout 的話，下游每隔幾秒吐一個 byte 就能讓請求無限期掛著，所以另外需要 request timeout。

**設定原則**

- 以下游延遲分佈為依據，通常取 p99 至 p99.9 再加餘量。
- 設太短會把正常的慢請求判為失敗，並觸發 retry 增加負載；設太長則資源佔用時間過久，失去保護效果。
- 上游的 timeout 必須大於下游的 timeout 加上重試的總時間，否則上游已放棄，下游還在做無效工作。

**Deadline propagation**：把「剩餘時間預算」隨請求往下傳（例如 gRPC deadline）。每一層收到請求時先檢查剩餘時間，不足就直接拒絕。

**兩個常被忽略的語意**

- Timeout 的結果是「未知」而不是「失敗」。請求可能已在下游執行完成，只是回應沒傳回來，這直接影響 retry 的安全性。
- 呼叫端 timeout 不代表下游停止工作，除非有 cancellation 傳遞機制（gRPC cancellation、HTTP/2 RST_STREAM、Go 的 `context`）。

## 2. Retry

**定義**：操作失敗後重新發出相同請求，用來處理暫時性故障（transient fault），如封包遺失、連線重置、下游實例重啟、短暫過載。

**可重試條件**

- 可重試：connection reset、timeout、HTTP 502/503/504、429（需遵守 `Retry-After`）。
- 不可重試：400、401、403、404、422 等由請求本身造成的錯誤，重試結果不會改變。

**Idempotency（冪等性）**：操作執行一次與多次的效果相同，即 f(f(x)) = f(x)。因為 timeout 的結果未知，重試非冪等操作可能造成重複執行（如重複扣款）。

- HTTP 語意上 GET、PUT、DELETE 為冪等，POST 不是。
- 非冪等操作要重試，需由 client 帶 idempotency key，server 以該 key 去重。

**Backoff 與 Jitter**

Exponential backoff，第 n 次重試前的等待時間：

$$delay(n) = min(cap, base · 2ⁿ)$$

只做 exponential backoff 的話，同時失敗的 client 會在相同時間點同時重試，負載呈現週期性尖峰。因此要加 jitter，常用的 full jitter 為：

delay(n) = random(0, min(cap, base · 2ⁿ))

**重試放大（retry amplification）**：呼叫鏈有 n 層、每層最多嘗試 k 次時，最底層最壞情況收到 kⁿ 倍請求。3 層各嘗試 3 次即 27 倍。下游過載時這會讓負載進一步上升，系統無法自行恢復（retry storm）。

對策：

- 限制最大嘗試次數，通常 2–3 次。
- 只在呼叫鏈的單一層做 retry。
- Retry budget：限制重試流量佔總流量的比例（例如 ≤ 10%），超過就不再重試。gRPC retry throttling 與 Finagle 採用這種做法。

**成功率的前提**：單次失敗機率為 p 且各次獨立時，k 次全失敗的機率為 pᵏ。但下游過載時各次失敗高度相關，獨立假設不成立，此時 retry 無法提高成功率，只會增加負載。這是需要 circuit breaker 的原因。

## 3. Circuit Breaker

**定義**：包在遠端呼叫外層的狀態機，持續統計呼叫結果。失敗率超過門檻時，在一段時間內直接拒絕呼叫（fail fast），不再送往下游。

**狀態與轉移**

|狀態|行為|轉移條件|
|---|---|---|
|Closed|請求正常通過，統計結果|滑動視窗內失敗率 ≥ 門檻 → Open|
|Open|請求立即被拒絕，不呼叫下游|經過設定的等待時間 → Half-Open|
|Half-Open|只放行少量探測請求|探測失敗率低於門檻 → Closed；否則 → Open|

**主要參數**（以 Resilience4j 命名為例）

- `failureRateThreshold`：觸發 Open 的失敗率門檻。
- `slidingWindowType` / `slidingWindowSize`：統計視窗，分為 count-based（最近 N 次呼叫）與 time-based（最近 N 秒）。
- `minimumNumberOfCalls`：視窗內至少累積多少次呼叫才計算失敗率，避免低流量時 1 次失敗就等於 100%。
- `waitDurationInOpenState`：Open 狀態持續時間。
- `permittedNumberOfCallsInHalfOpenState`：Half-Open 放行的探測請求數。
- `slowCallRateThreshold` / `slowCallDurationThreshold`：把慢呼叫也計入觸發條件。

**失敗的認定**：計入 timeout、連線錯誤、5xx。4xx 屬於呼叫端錯誤，不反映下游健康狀態，不應計入。

**Fallback**：斷路器拒絕請求後的處理方式，包括回傳快取值、回傳預設值、降級功能，或直接回錯誤。

**粒度**：統計單位可以是整個服務、單一 endpoint 或單一 host。粒度太粗時，一個故障實例或一個故障分片會讓整個服務被斷開。

**名詞釐清**：Envoy 的「circuit breaking」實際上是連線數、pending request 數、並行請求數的上限，性質接近 bulkhead。Envoy 中對應上述狀態機行為的是 outlier detection（依錯誤率把單一 host 從負載平衡池中暫時剔除）。



```
整體 deadline
  └─ Retry
       └─ Circuit Breaker
            └─ 單次嘗試 Timeout
                 └─ 遠端呼叫
```

- 單次 timeout 在最內層：每次嘗試各自計時，timeout 被記為一次失敗。
- Circuit breaker 在 retry 內層：每次嘗試的結果都計入統計。
- Retry 在外層：斷路器 Open 時拋出的拒絕例外應設為不可重試，否則只是空轉。
- 整體 deadline 在最外層，約束條件為：

Σ（單次 timeout）+ Σ（backoff 等待）≤ 整體 deadline < 上游的 timeout

Resilience4j 的預設裝飾順序即為 Retry → CircuitBreaker → RateLimiter → TimeLimiter → Bulkhead。

**互補關係**：Timeout 產生失敗訊號，Retry 消化其中暫時性的部分，Circuit Breaker 在失敗變成持續性時切斷 Retry 造成的負載放大。只有 Timeout 加 Retry 而沒有 Circuit Breaker 或 retry budget 的系統，在下游過載時會自己加重故障。