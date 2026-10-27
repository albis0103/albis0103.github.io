
SQLAlchemy 是 Python 的 SQL 工具包與 ORM（Object-Relational Mapper）。它由兩個可獨立使用的層組成：**Core** 與 **ORM**。目前主流版本為 2.0 系列，採用 2.0 風格 API（`select()` + `Session.execute()/scalars()`，型別化的 `Mapped[]`宣告）。

## 架構

```
Application
    │
    ├─ ORM        : DeclarativeBase、Mapped[]、Session、relationship、unit of work、identity map
    │
    ├─ Core       : SQL Expression Language（select/insert/update/delete）、Table/MetaData、Connection
    │
    └─ Engine     : 連線池（Pool）、Dialect、DBAPI 抽象
         │
      DBAPI driver（psycopg / pymysql / sqlite3 ...）
```

|層|核心物件|職責|
|---|---|---|
|Engine|`Engine`, `Pool`, `Dialect`|建立連線、管理 connection pool、依 dialect 產生對應資料庫的 SQL|
|Core|`Table`, `MetaData`, `select()` 等|以 Python 表達式建構 SQL AST，編譯為參數化 SQL|
|ORM|`Session`, `DeclarativeBase`, `relationship()`|物件與列映射、變更追蹤（unit of work）、identity map、lazy/eager loading|

## Engine 與連線池

```python
from sqlalchemy import create_engine

engine = create_engine(
    "postgresql+psycopg://user:pw@host:5432/db",
    pool_size=5,          # 常駐連線數
    max_overflow=10,      # 超出 pool_size 的暫時連線上限
    pool_pre_ping=True,   # 取出連線前先檢測是否存活
    echo=False,
)
```

URL 格式為 `dialect+driver://user:password@host:port/database`。`Engine` 為 process 層級單例，不應對每個請求重建。

## Core：SQL Expression Language

```python
from sqlalchemy import MetaData, Table, Column, Integer, String, select

metadata = MetaData()
users = Table(
    "users", metadata,
    Column("id", Integer, primary_key=True),
    Column("name", String(50), nullable=False),
)

stmt = select(users.c.id, users.c.name).where(users.c.name == "jason")

with engine.connect() as conn:
    for row in conn.execute(stmt):
        print(row.id, row.name)
```

所有值皆以 bind parameter 傳遞，因此不會產生字串拼接式的 SQL injection。

## ORM（2.0 風格）
[[泛型型別標註 Mapped]]

```python
from sqlalchemy import ForeignKey, String, select
from sqlalchemy.orm import (
    DeclarativeBase, Mapped, mapped_column, relationship, Session, selectinload,
)

class Base(DeclarativeBase):
    pass

class User(Base):
    __tablename__ = "users"
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(50))
    posts: Mapped[list["Post"]] = relationship(back_populates="author")

class Post(Base):
    __tablename__ = "posts"
    id: Mapped[int] = mapped_column(primary_key=True)
    title: Mapped[str] = mapped_column(String(100))
    user_id: Mapped[int] = mapped_column(ForeignKey("users.id"))
    author: Mapped["User"] = relationship(back_populates="posts")

Base.metadata.create_all(engine)

with Session(engine) as session:
    session.add(User(name="jason", posts=[Post(title="hello")]))
    session.commit()

    stmt = select(User).options(selectinload(User.posts)).where(User.name == "jason")
    user = session.scalars(stmt).one()
```

## Session 的行為

- **Unit of Work**：`session.add()` 與屬性修改只記錄於 session，`flush()` 時才依相依順序產生 INSERT/UPDATE/DELETE；`commit()` 會先 flush 再提交 transaction。
- **Identity Map**：同一 session 內，同一主鍵只對應一個 Python 物件。
- **Autobegin**：2.0 中 session 在首次使用時自動開啟 transaction。
- **`expire_on_commit=True`（預設）**：commit 後物件屬性標記為過期，下次存取會重新查詢；detached 物件存取會拋 `DetachedInstanceError`。
- Session 非 thread-safe，應每個請求／執行緒各自建立。

## 關聯載入策略

|策略|作法|適用|
|---|---|---|
|`lazy="select"`（預設）|存取屬性時才發 SELECT|容易產生 N+1|
|`selectinload`|第二次查詢 `WHERE id IN (...)`|一對多、集合|
|`joinedload`|同一查詢 JOIN|多對一、一對一|
|`subqueryload`|子查詢|較少使用|

N+1 問題的處理方式是在查詢中以 `.options(...)` 明確指定載入策略。

## 非同步

```python
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession, async_sessionmaker

engine = create_async_engine("postgresql+asyncpg://user:pw@host/db")
SessionLocal = async_sessionmaker(engine, expire_on_commit=False)

async with SessionLocal() as session:
    result = await session.execute(select(User))
```

async 模式下 lazy load 無法隱式 await，須使用 `selectinload` 等 eager loading，或設定 `expire_on_commit=False`。

## Migration

SQLAlchemy 本身只提供 `create_all()`，不處理 schema 演進。版本化的 schema migration 通常搭配 **Alembic**（同作者維護），可由 model metadata 做 autogenerate 差異。

## 與 Raw SQL 混用

```python
from sqlalchemy import text
session.execute(text("SELECT * FROM users WHERE id = :id"), {"id": 1})
```


# `PRAGMA`

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