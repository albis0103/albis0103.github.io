原因
- 寫入策略 Race - Condition
	- **先更新 DB，再更新 Redis（Update DB -> Update Redis）**:DB 留著 B 的最新資料，但 Redis 卻寫入了 A 的舊資料。
	- **先刪除 Redis，再更新 DB（Delete Redis -> Update DB）**：Redis 中保存了舊資料，與 DB 不一致。
	        
	- **先更新 DB，再刪除 Redis（Cache Aside Pattern）**：最推薦方式
- Concurrency and Asynchronous Process
	- Master - Slave Replication latency
		-  A 更新 Master DB 後隨即刪除 Redis ，B 讀取快取 Miss，轉而向 **Slave DB** 讀取資料。若Master 到 Slave 的同步延遲， B 就會讀到 Slave 舊資料並寫回 Redis。
	- Asynchronous Process Latency
		- 用非同步（MQ, Binlog監聽, ...）更新Redis, 當系統異常（MQ積壓、網路波動、Consumer延遲）Cache資料會落後DB
- 系統異常
	- Uncommitted Transactions
		- In Spring, 若`@Transactional` method操作Redis, 可能發生Redis成功但是DB發生Exception Rollback
### Cache 寫入策略
- 先更新 DB，再更新 Cache 並發寫入時，DB 的寫入順序與 Redis 的寫入順序可能不同：
``` 
T1: UPDATE db SET v=1 
T2: UPDATE db SET v=2 
T2: SET cache v=2 
T1: SET cache v=1 ← cache=1, DB=2，持續不一致直到 TTL 
``` 
另外，寫多讀少時，每次更新都寫 cache 是浪費。**不建議。** 
- 先刪除 Cache，再更新 DB 
``` 
T1(寫): DEL cache 
T2(讀): GET cache → miss → SELECT db → 舊值 v0 T1(寫): UPDATE db SET v=1 
T2(讀): SET cache v0 ← cache=v0, DB=v1 
``` 
讀請求在刪除與 DB commit 的窗口內回填舊值，發生機率高。 

- 先更新 DB，再刪除 Cache（Cache-Aside 標準做法） 
``` 
T1(讀): GET cache → miss 
T1(讀): SELECT db → v0 
T2(寫): UPDATE db SET v=1 
T2(寫): DEL cache 
T1(讀): SET cache v0 ← cache=v0, DB=v1 
```



| **方案**                          | **運作機制**                                     | **優缺點**                                  |
| ------------------------------- | -------------------------------------------- | ---------------------------------------- |
| **Cache Aside + 設定 TTL**        | 寫入時先更新 DB，再刪除 Redis<br>每個 key 必須設定過期時間。      | 最常用、實作簡單，TTL 能作為不一致的保底(TTL=不一致上線時間)。     |
| **延遲雙刪（Delayed Double Delete）** | 先刪 Redis  更新 DB ，暫停一定時間（如 500ms），再刪一次 Redis。 | 解決「先刪 Redis 後更新 DB」的併發問題                 |
| **CDC / Binlog 訂閱**             | 透過 Canal 監聽 DB Binlog，非同步更新/刪除 Redis。        | 解耦業務代碼，保證 DB 成功後才處理快取；但存在微秒到毫秒級的最終一致性延遲。 |
| **分散式鎖（Distributed Lock）**      | 對強一致性要求極高的 key（如庫存），讀寫時透過 Redlock 加鎖。        | 徹底解決併發問題，但會大幅降低系統吞吐量。                    |

#### 延遲雙刪（Delayed Double Delete）

```
1. DEL cache
2. UPDATE db
3. sleep(N ms)     // N ≥ 一次讀請求「查 DB + 回填 cache」的耗時
4. DEL cache
```

第二次刪除清掉 (b) 情境下被回填的舊值。缺點：N 為經驗值，且第二次 DEL 仍可能失敗。第二次刪除應走非同步（MQ 或延遲佇列），避免阻塞請求。

#### 重試與訊息佇列保證刪除

DB commit 後把「刪除 cache」作為訊息投遞，消費者失敗則重試，直到成功：

```
UPDATE db → 發送 MQ(key) → consumer: DEL cache（失敗重試 / 進 DLQ）
```

需處理 consumer 冪等（DEL 本身冪等，天然滿足）。

####  訂閱 binlog 非同步失效（CDC）

透過 Canal / Debezium 訂閱 MySQL binlog，由獨立服務解析變更後刪除（或更新）對應 cache key。

- 業務程式碼不需處理 cache 失效，與寫入路徑解耦。
- 失效動作以 DB 實際 commit 為準，不會出現「cache 已刪但 DB 未 commit」。
- 重試與順序由 CDC pipeline 負責（同一 key 的事件依 binlog 順序處理）。
- 代價：新增元件、引入秒級以內的失效延遲。

#### 版本號 / 時間戳比對

cache 值帶 `version`（來自 DB 的版本欄位或 binlog position）。回填時以 Lua script 做 compare-and-set：

```lua
-- KEYS[1]=key, ARGV[1]=new_version, ARGV[2]=value
local cur = redis.call('HGET', KEYS[1], 'version')
if (not cur) or tonumber(ARGV[1]) > tonumber(cur) then
  redis.call('HSET', KEYS[1], 'version', ARGV[1], 'value', ARGV[2])
  return 1
end
return 0
```

可擋掉舊值覆蓋新值的回填。前提是 DB 端有單調遞增版本。

#### 3.6 分散式鎖 / 單飛（singleflight）

- 對同一 key 的 cache miss 回源加鎖，避免多個請求同時回填（同時緩解 cache stampede）。
- 寫路徑與回填路徑對同一 key 互斥，可消除 (c) 的殘餘競態，代價是吞吐下降與鎖本身的失效處理（鎖逾時、續租）。