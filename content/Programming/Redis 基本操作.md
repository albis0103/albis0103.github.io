- `redis-cli`：連線本地redis
- `redis-cli -h host -p port -a password`：遠程連線redis
- `AUTH password`：密碼驗證
- `PING`: 檢測是否啟動，連結正常
- `select index`：切換 Database of `index` 
- `exit`：退出 redis-cli

- `SET k v`/ `GET k`/ `DEL k`: key - value 操作
- `EXISTS k`：檢查是否存在
- `EXPIRE k sec`：設定ttl 過期時間
- `TTL k`：剩餘存活時間
- `KEYS pattern`找符合 pattern 的 key
```
SET runoob1 redis
SET runoob2 mysql
SET runoob3 mongodb
KEYS runoob*
1) "runoob3"
2) "runoob1"
3) "runoob2"
```

### Data Structure
### String
: Cache, counter, lock
- `SET k v [EX sec] [NX\XX]`
	- `EX sec` ：過期時間
	- `NX`: If Absent, `XX`: if existed
- `INCR k`: Increasing Key
- `MSET/MGET k1 v1 k2 v2`: batch set or get keys
- `DEL KEY`
### Hash
: Object field
- `HSET k f1 v1 f2 v2`
	- `HSET user:101 age:"25 email "alice@test.com"` 
- `HGET key field`/ `HGETALL k`: 
	- `HGET user:101 name`: "Alice"
- `HDEL user:1 age`
- `HINCRBY user:1 age 1`
- `HEXISTS user:1 name`

### List
: Queue
- `LPUSH k a b c`/ `RPUSH k d`
- `LPOP/RPOP k
- `LLEN k
- `LRANGE k start end`: range query
- `LINDEX k idx`: query k[0]