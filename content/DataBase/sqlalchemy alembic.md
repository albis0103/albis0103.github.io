# Alembic

Alembic 是 SQLAlchemy 作者維護的 **資料庫 schema migration（遷移）工具**。SQLAlchemy 本身只有 `create_all()`，只能建立「不存在的表」，不會修改既有的表。Alembic 負責 schema 隨時間演進：新增欄位、改型別、加索引、刪表等，並且有版本紀錄，可升級也可降級。

## 1. 要解決的問題

```python
class User(Base):
    __tablename__ = "users"
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(50))
    email: Mapped[str] = mapped_column(String(100))   # ← 後來新增的欄位
```

資料庫中 `users` 已存在，`create_all()` 看到表已存在就跳過，不會補 `email` 欄位。需要的是 `ALTER TABLE users ADD COLUMN email ...`，而且要能在 dev、staging、production 各環境一致地執行、並可回滾。這就是 Alembic 的工作。

## 2. 核心概念

|概念|說明|
|---|---|
|Migration script（revision）|`versions/` 下的一個 Python 檔，包含 `upgrade()` 與 `downgrade()` 兩個函式|
|Revision ID|每個 migration 的唯一識別碼（如 `a1b2c3d4e5f6`）|
|`down_revision`|指向前一個 revision，串成一條鏈（類似 git 的 parent commit）|
|`alembic_version` 表|資料庫中的一張表，只存一列：目前套用到的 revision ID|
|`head`|鏈上最新的 revision|
|`env.py`|Alembic 的執行環境設定，指定 engine 與 `target_metadata`|

版本鏈與資料庫狀態的關係：

```
base ── rev_A ── rev_B ── rev_C (head)
                   ▲
        alembic_version 表記錄 = rev_B  → 資料庫目前停在 rev_B，尚未套用 rev_C
```

## 3. 目錄結構

```
project/
├─ alembic.ini            # 設定檔（資料庫 URL、script 位置、logging）
└─ alembic/
   ├─ env.py              # 連線與 metadata 設定
   ├─ script.py.mako      # 產生 migration 檔的模板
   └─ versions/
       ├─ a1b2_create_users.py
       └─ c3d4_add_email_to_users.py
```

初始化：

```bash
alembic init alembic
```

## 4. 與 SQLAlchemy 的串接（`env.py`）

關鍵是讓 Alembic 看到你的 model metadata：

```python
# alembic/env.py
from app.db.base import Base
import app.models            # 必須 import，讓所有 model 註冊進 Base.metadata

target_metadata = Base.metadata
```

這對應前面討論過的問題：model 檔案沒被 import，`Base.metadata` 就不完整，autogenerate 會誤判這些表不存在或應被刪除。

資料庫 URL 設在 `alembic.ini` 的 `sqlalchemy.url`，或在 `env.py` 內從環境變數讀取。

## 5. Migration 檔的結構

```python
"""add email to users

Revision ID: c3d4e5f6a7b8
Revises: a1b2c3d4e5f6
"""
from alembic import op
import sqlalchemy as sa

revision = "c3d4e5f6a7b8"
down_revision = "a1b2c3d4e5f6"

def upgrade() -> None:
    op.add_column("users", sa.Column("email", sa.String(100), nullable=True))

def downgrade() -> None:
    op.drop_column("users", "email")
```

`op` 是 Alembic 的操作物件，提供 `create_table`、`drop_table`、`add_column`、`drop_column`、`alter_column`、`create_index`、`create_foreign_key`、`execute`（執行 raw SQL）等。

## 6. 常用指令

|指令|作用|
|---|---|
|`alembic revision -m "msg"`|建立空白 migration 檔，手寫內容|
|`alembic revision --autogenerate -m "msg"`|比對 `target_metadata` 與資料庫現況，自動產生 migration|
|`alembic upgrade head`|升級到最新|
|`alembic upgrade +1`|往前套用一個 revision|
|`alembic downgrade -1`|回退一個 revision|
|`alembic downgrade base`|回退到最初（所有 migration 之前）|
|`alembic current`|顯示資料庫目前的 revision|
|`alembic history`|顯示 revision 鏈|
|`alembic upgrade head --sql`|不連資料庫，輸出將執行的 SQL（offline mode）|
|`alembic stamp head`|只更新 `alembic_version`，不執行任何 migration（用於把既有資料庫標記為某版本）|

## 7. Autogenerate 的行為與限制

autogenerate 流程：

```
讀取 target_metadata（程式碼中 model 的定義）
        ↓  比對
反射資料庫現況（inspect 實際 schema）
        ↓
產生差異 → 寫成 upgrade()/downgrade() 內容
```

**可偵測：** 新增／刪除表、新增／刪除欄位、欄位 nullable 變更、索引與 unique constraint 的增刪、外鍵的增刪。

**無法可靠偵測，需人工檢查或手寫：**

|情況|原因|
|---|---|
|表或欄位**改名**|只看到「舊的消失、新的出現」，會產生 drop + add，造成**資料遺失**；應手改為 `op.rename_table` / `op.alter_column(..., new_column_name=...)`|
|欄位型別變更|預設不比對型別，需在 `env.py` 設定 `compare_type=True`|
|server_default 變更|需設定 `compare_server_default=True`|
|CHECK constraint、部分資料庫專屬物件|支援有限|
|資料遷移（data migration）|autogenerate 只處理 schema，資料搬移要自己用 `op.execute` 寫|

因此 autogenerate 的產物是**草稿**，套用前必須閱讀 migration 檔。

## 8. SQLite 的特殊限制

SQLite 的 `ALTER TABLE` 功能有限（早期版本不支援 drop column、改型別、加 constraint）。Alembic 的解法是 **batch mode**：建立新表、複製資料、刪舊表、改名。

```python
with op.batch_alter_table("users") as batch_op:
    batch_op.alter_column("name", type_=sa.String(100))
    batch_op.drop_column("old_col")
```

並在 `env.py` 的 `context.configure(...)` 加上 `render_as_batch=True`，autogenerate 就會自動產生 batch 寫法。

## 9. 典型工作流程

```
1. 修改 model（例如新增 email 欄位）
2. alembic revision --autogenerate -m "add email to users"
3. 閱讀並修正產生的 migration 檔
4. alembic upgrade head         ← 套用到本機資料庫
5. 將 migration 檔與 model 變更一起 commit 進版本控制
6. 部署時在目標環境執行 alembic upgrade head
```

migration 檔視為程式碼的一部分，與 model 變更同一個 commit。

## 10. 常見問題

|現象|原因與處理|
|---|---|
|autogenerate 產生空的 migration|model 沒被 import 進 `env.py`，或資料庫已與 model 一致|
|autogenerate 想刪掉所有表|`target_metadata` 是空的（沒 import models）|
|`Can't locate revision`|`alembic_version` 記錄的 ID 在 `versions/` 中找不到（檔案被刪或分支不同）|
|多個 head（`Multiple head revisions`）|兩人從同一 revision 各自建立 migration，需用 `alembic merge heads` 合併|
|既有資料庫導入 Alembic|先 autogenerate 出對應現況的初始 migration，再對資料庫執行 `alembic stamp head`，避免重複建表|

## 摘要

Alembic 是版本化的 schema migration 工具：每個 migration 是含 `upgrade()` / `downgrade()` 的 Python 檔，以 `down_revision`串成鏈，資料庫中的 `alembic_version` 表記錄目前版本。`env.py` 的 `target_metadata = Base.metadata` 是與 SQLAlchemy 的銜接點；`--autogenerate` 可由 model 與資料庫的差異產生草稿，但改名、型別變更與資料遷移需人工處理。SQLite 需使用 batch mode。