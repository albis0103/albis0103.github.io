
`PRAGMA` 是 **SQLite 專有的指令**，用來查詢或修改 SQLite 引擎的內部設定與行為。它不是標準 SQL，PostgreSQL、MySQL 沒有這個語法。

```sql
PRAGMA foreign_keys = ON;      -- 設定
PRAGMA foreign_keys;           -- 查詢目前值
```

## 1. 為什麼在 SQLAlchemy 會遇到

SQLite 有幾個預設行為與其他資料庫不同，需要用 PRAGMA 調整，其中最常見的是 **外鍵約束預設關閉**：

```sql
PRAGMA foreign_keys = ON;
```

沒開的話，即使 model 寫了 `ForeignKey("users.id")`，SQLite 也不會檢查，可以插入指向不存在列的 `user_id`，`ON DELETE CASCADE` 也不會生效。

## 2. 作用範圍：每條連線

`foreign_keys` 是 **per-connection** 設定，不是寫進資料庫檔案。每次新建連線都要重新設定，所以不能只執行一次，必須在每條連線建立時執行。SQLAlchemy 的做法是監聽 `connect` 事件：

```python
from sqlalchemy import create_engine, event

engine = create_engine("sqlite:///app.db")

@event.listens_for(engine, "connect")
def set_sqlite_pragma(dbapi_connection, connection_record):
    cursor = dbapi_connection.cursor()
    cursor.execute("PRAGMA foreign_keys=ON")
    cursor.close()
```

`connect` 事件在 pool 建立新的 DBAPI 連線時觸發，因此每條連線都會套用。

## 3. 常用 PRAGMA

|PRAGMA|作用|持久性|
|---|---|---|
|`foreign_keys = ON`|啟用外鍵約束|每連線|
|`journal_mode = WAL`|改用 Write-Ahead Logging，讀寫可並行，併發表現較好|寫入資料庫檔案，持久|
|`synchronous = NORMAL`|降低 fsync 頻率，配合 WAL 提升寫入效能|每連線|
|`busy_timeout = 5000`|遇到鎖時等待最多 5000 ms，而非立刻拋 `database is locked`|每連線|
|`cache_size = -64000`|頁面快取大小（負值單位為 KiB，此例約 64 MB）|每連線|
|`table_info(users)`|查詢某表的欄位定義|查詢用|
|`index_list(users)`|查詢某表的索引|查詢用|
|`foreign_key_list(posts)`|查詢某表的外鍵|查詢用|

## 4. 常見的 SQLite + SQLAlchemy 設定組合

```python
@event.listens_for(engine, "connect")
def set_sqlite_pragma(dbapi_connection, connection_record):
    cursor = dbapi_connection.cursor()
    cursor.execute("PRAGMA foreign_keys=ON")
    cursor.execute("PRAGMA journal_mode=WAL")
    cursor.execute("PRAGMA synchronous=NORMAL")
    cursor.execute("PRAGMA busy_timeout=5000")
    cursor.close()
```

## 5. 與前一則交易管理的關聯

pysqlite（Python 內建 `sqlite3`）對交易的處理有已知問題：預設不會在 `BEGIN` 前正確發出語句，導致 SAVEPOINT 與部分隔離行為不可靠。這也是前面提到「SQLite 需額外設定」的原因。SQLAlchemy 文件提供的解法是同樣用事件，關閉 pysqlite 自動管理交易，改由 SQLAlchemy 發出 `BEGIN`：

```python
@event.listens_for(engine, "connect")
def do_connect(dbapi_connection, connection_record):
    dbapi_connection.isolation_level = None      # 停用 pysqlite 自己的 BEGIN 處理

@event.listens_for(engine, "begin")
def do_begin(conn):
    conn.exec_driver_sql("BEGIN")
```

## 6. 直接執行 PRAGMA 查詢

```python
from sqlalchemy import text

with engine.connect() as conn:
    print(conn.execute(text("PRAGMA foreign_keys")).scalar())   # 0 或 1
    print(conn.execute(text("PRAGMA table_info(users)")).all())
```

## 摘要

`PRAGMA` 是 SQLite 的引擎設定指令。在 SQLAlchemy 中，最重要的用途是啟用外鍵約束（`foreign_keys=ON`），而且因為它是 per-connection 設定，必須透過 `connect` 事件在每條新連線上執行。其他常用的有 WAL 模式、`busy_timeout`，用於改善併發與鎖定行為。