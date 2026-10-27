
## 一、三者比較

|項目|sysbench|TPC-C|TPC-H|
|---|---|---|---|
|性質|開源工具，非標準規範|TPC 官方規範（OLTP）|TPC 官方規範（OLAP/決策支援）|
|負載|合成的單表/少數表操作|批發商訂單系統，5 種交易混合|22 條複雜 ad-hoc 查詢|
|Schema|`sbtest1..N`，每表 (id, k, c, pad)|9 張表（Warehouse, District, Customer, Order, New-Order, Order-Line, Stock, Item, History）|8 張表（LINEITEM, ORDERS, CUSTOMER, PART, PARTSUPP, SUPPLIER, NATION, REGION）|
|存取型態|點查詢、範圍查詢、單列 UPDATE/INSERT/DELETE|短交易、多表 read-write、高併發、有鎖競爭|大範圍掃描、多表 join、聚合、group by、排序|
|規模參數|`--tables`、`--table-size`|warehouse 數（W）|Scale Factor（SF，1 ≈ 1GB 原始資料）|
|瓶頸|CPU、lock、log、buffer pool|commit latency、lock contention、WAL/redo fsync、I/O|掃描頻寬、join/aggregate 執行器、並行度、optimizer 計畫品質|
|官方認證|無|有（需審計）|有（需審計）|

## 二、sysbench

`oltp_read_write` 等內建 Lua 腳本，三階段：

```bash
sysbench oltp_read_write --mysql-host=... --tables=16 --table-size=1000000 prepare
sysbench oltp_read_write --threads=64 --time=300 --report-interval=10 run
sysbench oltp_read_write ... cleanup
```

- 每個交易預設含 10 次點查、4 次範圍查詢（simple/sum/order/distinct）、2 次 UPDATE（index/non-index）、1 次 DELETE + 1 次 INSERT。
- 其他腳本：`oltp_read_only`、`oltp_point_select`、`oltp_write_only`、`oltp_insert`、`oltp_update_index`。
- 指標：TPS、QPS、latency（avg/95th percentile/max）、errors/s、reconnects/s。
- 變因：`--threads`（併發度）、`--rand-type`（uniform/zipfian，控制熱點）、資料量與 buffer pool 的比例。
- 限制：沒有業務語意，資料分布簡單，無多表 join，不能代表真實應用；適合做相對比較（調參、硬體比較）。

## 三、TPC-C（OLTP）

**模型**：W 個 warehouse，每個 warehouse 10 個 district，每個 district 3000 位 customer。

**五種交易及其比例（最低占比）**：

|交易|占比|特性|
|---|---|---|
|New-Order|約 45%|讀寫，主要吞吐指標，平均每筆約 10 個 order-line|
|Payment|約 43%|讀寫，更新 warehouse/district/customer 的 YTD 並寫 History|
|Order-Status|4%|唯讀|
|Delivery|4%|批次式讀寫，每次處理 10 個 district，可延遲執行|
|Stock-Level|4%|唯讀，掃描近 20 筆訂單的庫存|

**測量方式**：

- 模擬終端：每個 warehouse 對應 10 個 terminal，每個 terminal 依序執行「選擇交易 → 輸入 → 送出 → 取得回應 → think time」。
- 有 keying time 與 think time（規範定義），所以吞吐量受 W 限制，每個 warehouse 的 tpmC 上限約 12.86。要提高 tpmC 必須增加 W 而不是單純增加執行緒。
- **主要指標 tpmC**：每分鐘完成的 New-Order 交易數（measurement interval 內，穩態）。
- 反應時間限制：各交易 90th percentile 須低於規範上限（如 New-Order 5 秒）。
- 另有 ACID 測試、資料分布規則（例如 1% 的 New-Order 跨 warehouse、1% 回滾）。
- 價格效能：$/tpmC。

**開源實作**：HammerDB、BenchBase（原 OLTP-Bench）、percona sysbench-tpcc、go-tpc。這些通常移除 think time，因此結果稱 tpmC-like，不能直接與官方認證結果比較。

## 四、TPC-H（OLAP）

**資料生成**：`dbgen -s <SF>` 產生資料，`qgen` 產生 22 條查詢（含隨機參數）。SF 常用 1、10、100、300、1000。

**測試流程**（官方）：

1. **Load Test**：載入資料、建索引、收統計資訊。
2. **Power Test**：單一 stream，依序執行 RF1（插入新訂單）→ 22 條查詢 → RF2（刪除舊訂單）。
3. **Throughput Test**：S 個並行 query stream（S 依 SF 決定），加上一個 refresh stream 執行 RF1/RF2。

**指標**：

- Power@Size = 3600 × SF ÷ (∏ᵢ₌₁²² QI(i,0) × ∏ⱼ₌₁² RI(j,0))^(1/24)，即 24 個時間（22 條查詢 + 2 個 refresh）的幾何平均倒數。
- Throughput@Size = (S × 22 × 3600) ÷ Tₛ × SF，Tₛ 為 throughput test 的總耗時。
- **QphH@Size = √(Power@Size × Throughput@Size)**。
- 價格效能：$/QphH。

簡易做法（研究常用）：只跑 22 條查詢，量每條的 wall-clock 時間與總時間，不計算 QphH。

## 五、OLTP vs OLAP 測量重點對照

| |OLTP|OLAP|
|---|---|---|
|主指標|吞吐（TPS/tpmC）+ 延遲分布（p95/p99）|查詢完成時間、QphH|
|併發|數十到數千連線|通常數個到數十個 stream|
|單次操作|毫秒級|秒到分鐘級|
|資料集|熱資料可放進記憶體（受 W 影響）|通常大於記憶體，考驗 I/O 與掃描|
|關鍵變因|buffer pool、WAL/fsync、鎖、索引|並行度、記憶體（work_mem）、統計資訊、join 策略、列式/行式儲存|

