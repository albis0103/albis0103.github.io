
Execute Lua Script in redis: `EVAL script numkeys key arg`
ex.
```redis
EVAL "return redis.call('GET',KEYS[1])" 1 mykey
```
- `KEYS[]`: Key name from Lus(1 - based)
- `ARGV[]`: other parameter

- `redis.call(cmd, ...)`: 錯誤時向 client return error
- `redis.pcall(cmd, ...)`: return error table , 由Lua自行處理


### 型別轉換

|Redis → Lua|Lua → Redis|
|---|---|
|Integer reply → number|number → Integer（**小數會被截斷**）|
|Bulk string → string|string → Bulk string|
|Multi-bulk → table|table → Multi-bulk（遇第一個 `nil` 即截斷）|
|Nil bulk → `false`|`false` → Nil bulk|

**Lua 的 `nil` 與 Redis 的 nil reply 對應到 `false`**

### EVALSHA 與腳本快取

- `SCRIPT LOAD script` 回傳 SHA1；`EVALSHA sha1 numkeys ...` 只傳 hash，省頻寬
- 腳本快取於記憶體，**重啟後消失**，client 收到 `NOSCRIPT` error 重傳 `EVAL<lua_script>`


### 限制與風險

- **阻塞**：長時間腳本會阻塞整個實例。超過 `busy-reply-threshold`（預設 5000ms，原名 `lua-time-limit`）後，其他 client 收到 `BUSY` 錯誤，僅 `SCRIPT KILL`（腳本尚未寫入時）或 `SHUTDOWN NOSAVE` 可處理
- **確定性**：Redis 5.0+ 預設採 effects replication（複寫實際寫入命令而非腳本本身），非確定性函式限制已放寬，但腳本仍應避免依賴時間、隨機數

## 經典應用

### Mutex Lock safety release

problem:
- state: `KEYS[1] = 'tagA'` , A 執行很久, Lock 被 Redis刪除（自動過期）
- B 搶到 Lock, `KEYS[1] = 'tagB'` 
- A 執行完回來把 B的Lock刪掉
```lua
if redis.call('GET', KEYS[1] == ARGV[1]) then
	return redis.call('DEL', KEYS[1])
else
	return 0
end
```

### Watching Dog
: 當Lock 自動過期, Watching Dog 發現業務執行中，跟Redis請求鎖續期
```lua
if redis.call('GET', KEYS[1] == ARGV[1]) then
	return redis.call('PEXPIRE', KEYS[1], ARGV[2])
else 
	return 0
end
```


### Fixed Window Rate Limit
: count 當前請求數 限制流量
```lua
local cnt = redis.call('INCR', KEYS[1]) --遞增
if cnt == 1 then 
	redis.call('PEXPIRE', KEYS[1], ARGV[1])
end
if cnt > tonumber(ARGV[2]) then 
	return 0 --超過上限，拒絕
else
	return 1 --放行
```

### ZSET Sliding Window Rate Limiting
: 透過 Redis有序集合(ZSet)實現連續時間軸的Slinding window解決 Fixed window的Boundary Problem（限制一分鐘100次。0:59發送100次，1:01發送100次，兩秒內承受200次）
```lua
local now, window, limit = tonumber(ARGV[1]), tonumber(ARGV[2]), tonumber(ARGV[3])

--remove expire element
redis.call('ZREMRAGEBYSCORE', KSYS[1], 0, now - window)

--count
if redis.call('ZCARD', KEYS[1]) < limit then
	redis.call('ZADD', KEYS[1], now, ARGV[4])
	redix.call('PEXPIRE', KEYS[1], window)
	return 1
end
return 0
```
- **`ARGV[1]` (now)**：當前時間戳記（通常為毫秒，如 `1700000000000`）。
- **`ARGV[2]` (window)**：滑動視窗寬度（毫秒），例如 60000。
- **`ARGV[3]` (limit)**：視窗內允許的最大請求數。
- **`ARGV[4]`**：唯一識別識別碼（例如帶有毫秒+隨機數的 UUID），避免同毫秒多筆請求在 ZSET 中被覆蓋。

### 電商 Over-selling
: 檢查與扣減庫存在Application(Java/ Python)或Relation DB(MySQL)有 Lock Contention問題
假設目前庫存只剩 **1** 個，此時有兩個請求同時到達：

```
[線程 A] ──GET stock (拿到 1) ───────────────────────► DECRBY 1 (庫存變 0)
                                 ↑ 隙縫！
[線程 B] ───────────────GET stock (也拿到 1) ─────────► DECRBY 1 (庫存變 -1！超賣！)
```

**Basic: Prevent Over-selling**
```lua
local stock = tonumber(redis.call('GET', KEYS[1]) or '-1')
if stock >= tonumber(ARGV[1]) then
	redis.call('DECRBY', KEYS[1], ARGV[1])
	return 1 --扣減成功
end 
return 0 -- stock not enough
```
- **`KEYS[1]`**：商品的庫存 Key（例如 `product:1001:stock`）。
- **`ARGV[1]`**：預計要購買/扣減的數量（例如 `1` 或 `2`）。
- **`tonumber(... or '-1')`**：從 Redis 取出字串並轉為數字；若 Key 不存在或尚未預熱，預設為 `-1`。
**Advanced: Prevent Duplicate Claims**
- 限制每個User 只能買一次
```lua
local stockKey = KEYS[1] --商品庫存key(ex. product:1001:stock -> '100')
local userSetKey = KEYS[2] --搶購成功用戶的集合(ex. product:1001:users -> {"userA", ..} -> 0 or 1)

local userId = ARGV[1] --UserID
local buyNum = tonumber(ARGV[2]) --購買數量

-- 1. user 買過？
if redis.call('SISMEMBER', userSetKey, userId) == 1 then
	return -1
end
-- 2. stock enough?
local stock = tonumber(redis.call('GET', stockKey) or '0')
if stock < buyNum then 
	return 0 -- stock not enough
	
-- 3. 紀錄user跟扣紀錄
redis.call('DECRBY', stockKey, buyNum)
redis.call('SADD', userSetKey, userId)

return 1 --success
```

- **`SISMEMBER` 檢查**：使用 Redis 的 Set 集合（`SSET`）檢查當前 `userId` 是否已在(IsMember)「成功搶購名單」中。
- **`SADD` 記錄**：若庫存足夠，在扣減庫存的同時，將 `userId` 加入 Set 集合中。