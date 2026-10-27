
# `Session`：用來查詢

`Session` 是 SQLAlchemy ORM 的**工作單元（Unit of Work）管理者**：它持有一個 transaction、追蹤你載入或新增的 ORM 物件，並負責把物件的變更同步成 SQL 送到資料庫。前面談到的交易、`Mapped[]`、`Base` 最後都是透過 Session 操作。

## 1. Session 持有什麼

```
Session
 ├─ 一個 transaction（autobegin，必要時才從 engine 的 pool 取連線）
 ├─ identity map：{(class, primary key): 物件}
 └─ 變更追蹤：new / dirty / deleted 的物件集合
```

|組成|作用|
|---|---|
|Transaction|所有 SQL 在同一個交易內執行，`commit()` / `rollback()` 結束|
|Identity map|同一 session 內，同一主鍵只對應一個 Python 物件|
|變更追蹤|屬性被修改的物件（dirty）會在 `flush()` 時產生 UPDATE|

## 2. 基本操作

```python
from sqlalchemy import select
from sqlalchemy.orm import Session

with Session(engine) as session:
    # 新增
    user = User(name="jason")
    session.add(user)
    session.commit()

    # 查詢
    stmt = select(User).where(User.name == "jason")
    user = session.scalars(stmt).one()       # 回傳單一 User；0 或多筆會拋例外

    # 修改：直接改屬性，session 會偵測
    user.name = "jason wu"
    session.commit()                         # 產生 UPDATE

    # 刪除
    session.delete(user)
    session.commit()

    # 依主鍵取得（先查 identity map，沒有才發 SELECT）
    user = session.get(User, 1)
```

## 3. 查詢結果的取用方式

|方法|回傳|
|---|---|
|`session.scalars(stmt)`|`ScalarResult`，每列取第一欄（單一 ORM 物件時即該物件）|
|`session.scalars(stmt).all()`|`list[User]`|
|`.one()`|恰好一筆，否則拋 `NoResultFound` / `MultipleResultsFound`|
|`.first()`|第一筆或 `None`|
|`.one_or_none()`|0 或 1 筆；多筆拋例外|
|`session.execute(stmt)`|`Result`，每列是 `Row`（多欄位查詢時用）|

## 4. 物件的狀態（Object States）

ORM 物件相對於 Session 有五種狀態：

|狀態|意義|
|---|---|
|Transient|剛建立，未加入任何 session，無主鍵身分|
|Pending|已 `add()`，尚未 flush，還沒有 INSERT|
|Persistent|已在 session 內且對應資料庫中的一列（flush 後或查詢載入）|
|Detached|曾屬於 session，session 已關閉或物件被 `expunge`，仍有主鍵但不再被追蹤|
|Deleted|已 flush 了 DELETE，等待 commit|

```python
user = User(name="a")        # Transient
session.add(user)            # Pending
session.flush()              # Persistent（已 INSERT，在交易內）
session.commit()             # 仍 Persistent，但屬性過期（expire_on_commit）
session.close()              # Detached
```

`DetachedInstanceError` 就是對 Detached 物件存取過期或尚未載入的屬性所致。

## 5. `flush` 與 `commit` 的區別

| |`flush()`|`commit()`|
|---|---|---|
|發出 SQL|是|先 flush，再 COMMIT|
|結束 transaction|否|是|
|其他連線可見|否（未提交）|是|
|能 rollback|是|否|

flush 後可取得資料庫產生的主鍵：

```python
session.add(user)
session.flush()
print(user.id)      # 已有值，但交易尚未提交
```

`autoflush=True`（預設）時，執行查詢前會自動 flush，所以同一 session 內查詢看得到自己尚未 commit 的變更。

## 6. 建立 Session 的方式

```python
# 直接建立
session = Session(engine)

# 工廠：統一設定，之後重複產生
SessionLocal = sessionmaker(bind=engine, expire_on_commit=False)
with SessionLocal() as session:
    ...
```

`sessionmaker` 是產生 Session 的工廠，用來集中設定 `bind`、`autoflush`、`expire_on_commit` 等參數，不是 Session 本身。

## 7. 生命週期與執行緒

- Session 應對應**一個工作單元**（例如一個 HTTP 請求、一個 job），結束即 `close()`。
- Session **不是 thread-safe**，不可跨執行緒共用；每個執行緒或請求建立各自的 Session。
- Session 不是長生命週期的全域物件：identity map 會持續累積物件，長期存活會佔用記憶體並讀到過期資料。

## 8. Session 與 Engine 的關係

```
Engine（process 內唯一）──管理──▶ Pool ──▶ DBAPI 連線
   ▲
   │ bind
Session（每個工作單元一個）
```

Engine 是長期存在的資源（連線池），Session 是短期的工作單元，從 Engine 的 pool 借連線，交易結束後歸還。

## 摘要

`Session` 是 ORM 與資料庫之間的工作單元：它追蹤物件狀態（transient → pending → persistent → detached）、在 `flush()` 時把變更轉成 SQL、在 `commit()` 時結束交易。使用上，每個工作單元建立一個 Session，用 `select()` 搭配 `scalars()` 查詢，修改屬性即由 Session 偵測變更，結束時關閉。