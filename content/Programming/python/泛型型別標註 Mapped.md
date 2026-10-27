# `Mapped[]` 是什麼

`Mapped[T]` 是 SQLAlchemy 2.0 提供的**泛型型別標註（generic type annotation）**，用來宣告「這個 class 屬性對應到資料表的一個欄位，其 Python 型別為 `T`」。

## 1. 先看舊寫法與新寫法的差異

舊寫法（1.x）：

```python
class User(Base):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True)
    name = Column(String(50), nullable=False)
```

新寫法（2.0）：

```python
class User(Base):
    __tablename__ = "users"
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(50))
```

語法上，`id: Mapped[int] = mapped_column(...)` 由兩部分組成：

|部分|語法角色|作用|
|---|---|---|
|`id: Mapped[int]`|型別標註（annotation）|告訴型別檢查器與 SQLAlchemy：此屬性的 Python 型別是 `int`|
|`= mapped_column(...)`|賦值（右側的值）|提供欄位的額外設定（主鍵、長度、外鍵、index 等）|

## 2. 為什麼需要 `Mapped[]`

舊寫法中 `id = Column(Integer)` 對型別檢查器（mypy、Pyright）而言，`User.id` 是 `Column` 物件，但 `user.id`（實例屬性）其實是 `int`，兩者型別不同，靜態分析無法正確推斷。

`Mapped[int]` 解決這件事，它利用 descriptor 的概念區分兩種存取：

```python
User.id      # 類別層級 → InstrumentedAttribute[int]（可用於 SQL 表達式：User.id == 1）
user.id      # 實例層級 → int
```

因此：

```python
select(User).where(User.id == 1)   # 型別檢查器知道 User.id 是 SQL 表達式
user.id + 1                        # 型別檢查器知道 user.id 是 int
```

## 3. 型別如何對應到 SQL 型別

SQLAlchemy 會讀取 `Mapped[...]` 內的 Python 型別，**自動推導 SQL 欄位型別**，因此不需重複寫 `Integer`：

|Python 型別|預設 SQL 型別|
|---|---|
|`Mapped[int]`|`INTEGER`|
|`Mapped[str]`|`VARCHAR`（無長度）|
|`Mapped[float]`|`FLOAT`|
|`Mapped[bool]`|`BOOLEAN`|
|`Mapped[datetime]`|`DATETIME` / `TIMESTAMP`|

需要指定長度等細節時，才在 `mapped_column()` 內明確給 SQL 型別：

```python
name: Mapped[str] = mapped_column(String(50))
```

## 4. `Optional` 決定 nullable

`Mapped[]` 內的型別是否可為 `None`，直接決定欄位是否 `NULL`：

```python
from typing import Optional

name:  Mapped[str]            = mapped_column()   # NOT NULL
email: Mapped[Optional[str]]  = mapped_column()   # NULL（等同 Mapped[str | None]）
```

不需再另外寫 `nullable=False`。

## 5. 用在 relationship

`relationship` 同樣用 `Mapped[]` 標註，但內部型別是**關聯的 ORM class**：

```python
class User(Base):
    posts: Mapped[list["Post"]] = relationship(back_populates="author")   # 一對多 → list

class Post(Base):
    author: Mapped["User"] = relationship(back_populates="posts")         # 多對一 → 單一物件
```

- `Mapped[list["Post"]]`：集合（一對多），`user.posts` 是 `list[Post]`。
- `Mapped["User"]`：純量（多對一），`post.author` 是 `User`。
- `"Post"` 以字串寫是 forward reference，因為 `Post` 在此時尚未定義。

SQLAlchemy 從 `list[...]` 或單一型別判斷 relationship 的基數（cardinality）。

## 6. 完整對照

```python
class Post(Base):
    __tablename__ = "posts"

    id:      Mapped[int]  = mapped_column(primary_key=True)
    title:   Mapped[str]  = mapped_column(String(100))
    user_id: Mapped[int]  = mapped_column(ForeignKey("users.id"))
    author:  Mapped["User"] = relationship(back_populates="posts")
```

等價的 DDL：

```sql
CREATE TABLE posts (
    id      INTEGER      NOT NULL PRIMARY KEY,
    title   VARCHAR(100) NOT NULL,
    user_id INTEGER      NOT NULL,
    FOREIGN KEY(user_id) REFERENCES users (id)
);
```

注意 `author` 是 relationship，不對應資料表欄位，只有 `user_id` 是實際欄位。

## 摘要

`Mapped[T]` 只是型別標註，作用有三：(1) 讓靜態型別檢查正確區分類別層級與實例層級的存取；(2) 讓 SQLAlchemy 從 `T` 推導 SQL 型別與 nullability；(3) 在 relationship 中表示基數。實際的欄位設定仍由右側的 `mapped_column()` / `relationship()` 提供。