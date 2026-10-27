

## 環境建置

拉到最新程式碼後，依序執行下列指令，最後會得到一個有資料的 `joycare.db`。所有指令都在 `backend/` 資料夾內執行，因為資料庫網址 `sqlite:///./joycare.db` 是相對於執行目錄的，在其他目錄執行會建出另一個空資料庫。

```bash
# 在 repo 根目錄
python -m venv .venv
source .venv/bin/activate        # Windows：.venv\Scripts\activate
cd backend
pip install -r requirements.txt
alembic upgrade head            # 建 9 張表
python -m app.seed              # 寫入開發資料
```

確認成功：

```bash
python -c "
import sqlite3
c = sqlite3.connect('joycare.db')
for t in ['users','care_recipients','caregiver_profiles']:
    print(t, c.execute(f'select count(*) from {t}').fetchone()[0])
"
```

預期輸出是 `users 8`、`care_recipients 2`、`caregiver_profiles 5`。

- **重置資料庫：** 刪掉 `joycare.db`（若有也刪 `joycare.db-wal`、`joycare.db-shm`），再重跑 `alembic upgrade head` 與 `python -m app.seed`。
- **`.db-wal`、`.db-shm` 沒出現是正常的：** 這兩個檔只在連線開著時存在，最後一條連線關閉後 SQLite 會自動刪掉。
- **repo 路徑：** 不要放在 OneDrive 等同步資料夾內，同步程式可能鎖住 WAL 暫存檔（文件 AD-18）。
- **Python 版本：** 目前在 Python 3.11 驗證過。

## 目錄結構與分層規則

程式分成四層，呼叫方向只能由上往下：`api`（Controller）→ `services` → `domain`（純函式）與 `repositories` → `models`。

```text
backend/
  app/
    models/          ORM 模型
      base.py        Base、JSONList、JSONDict、utcnow
      account.py     User、CareRecipient、CaregiverProfile
      assessment.py  Question、Assessment
      recommendation.py  Recommendation
      booking.py     Booking
      schedule.py    ScheduleEvent
      feedback.py    Feedback
    core/
      database.py    Database（ReadSession、WriteSession）
      security.py    hash_password、verify_password
    domain/          列舉、字典代碼、純函式
      enums.py  slots.py  districts.py  tags.py
    seed.py          開發資料
    api/  services/  repositories/  schemas/    各模組自行新增
  alembic/           遷移
```

新增資料夾時要放 `__init__.py`，否則 `from app.xxx import ...` 會找不到模組。

|層|可以做|不可以做|
|---|---|---|
|Controller（api）|接收請求、驗證輸入、呼叫 Service|碰資料庫、寫商業規則|
|Service|串起流程、開關交易、commit|直接寫 SQL|
|domain|純函式、列舉、字典、計分|匯入資料庫或框架|
|Repository|查詢與寫入資料（flush）|commit|

三條規則要特別守：

1. **只有 Service 可以 commit。** Repository 與 Controller 都不可以（文件 AD-2）。
2. **每張表只有一個 Service 寫入。** 其他模組要改這張表，要透過那個 Service 提供的方法（AD-15）。
3. **商業規則只放 `domain/`。** 這層不依賴資料庫與框架，才能用純單元測試驗證（AD-1）。

## 資料庫存取

查詢用 `ReadSession`，會寫入的操作一律用 `WriteSession`。這不是建議，是為了避免「先檢查再寫入」被插隊，例如同一位照服員同時被兩個預約送出。

|Session|進交易時執行|用途|
|---|---|---|
|`ReadSession`|`BEGIN`|只查詢|
|`WriteSession`|`BEGIN IMMEDIATE`|任何會 INSERT、UPDATE、DELETE 的流程|

實驗結果：6 個連線同時做「先查再寫」，預設設定寫入 6 筆（超額）；`BEGIN` 的結果是 1 筆成功、其餘 5 筆報 `database is locked`；`BEGIN IMMEDIATE` 的結果是 1 筆成功、其餘 5 筆乾淨地被拒絕。

### 查詢

```python
from sqlalchemy import select
from app.core.database import Database
from app.domain.enums import UserRole
from app.models import User

db = Database("sqlite:///./joycare.db")

with db.ReadSession() as session:
    caregivers = session.scalars(
        select(User).where(User.role == UserRole.CAREGIVER)
    ).all()
```

### 寫入（先檢查再寫）

```python
from app.domain.enums import BookingStatus
from app.models import Booking

with db.WriteSession() as session:
    exists = session.scalar(
        select(Booking).where(Booking.rec_id == rec_id)
    )
    if exists:
        raise ValueError("這筆推薦已有預約")     # 實際要改成 AppError
    session.add(Booking(
        rec_id=rec_id, recipient_id=recipient_id, caregiver_id=caregiver_id,
        start_date=start, end_date=end, time_slot=slots,
        status=BookingStatus.PENDING,
    ))
    session.commit()                                  # 只有 Service 可以寫這行
```

### 使用時的注意事項

- **WriteSession 一開始就會鎖住整個資料庫的寫入。** 從第一個語句到 commit 之間不要做慢的事，例如呼叫外部服務、等使用者輸入。鎖等待上限是 5 秒，超過會報 `database is locked`。
- **不要用 ReadSession 寫資料。** 它沒有寫入鎖，同時寫會互相衝突。
- **外鍵已啟用。** 寫入不存在的 `user_id`、刪除仍被參照的列，都會直接報 `IntegrityError`。
- **JSON 欄位要用 `JSONList`、`JSONDict`。** 這兩種型別修改內容（`append`、設定 key）會被偵測並存檔；直接用 `JSON` 會漏存。
- **`expire_on_commit=False`。** commit 後物件仍可讀取欄位，不會重新查詢。
- **時間以 UTC 儲存。** SQLite 取回的 `datetime` 沒有時區，視為 UTC。
- **欄位長度不會被資料庫檢查。** `String(50)` 在 SQLite 不會拒絕更長的字串，要在輸入驗證層擋下。

## 字典代碼

狀態、角色、時段、行政區、標籤都只能用 `app/domain/` 裡定義的代碼，不要在程式裡自己寫字串。資料庫只存代碼（英文 slug），顯示用的中文名稱由前端或字典提供。

|檔案|內容|狀態|
|---|---|---|
|`enums.py`|`UserRole`、`BookingStatus` 等列舉|`BookingStatus` 為 pending／confirmed／rejected／in_progress／ended／cancelled|
|`slots.py`|21 個時段代碼，格式為 `<星期>_<時段>`，如 `MON_AM`、`SUN_EV`|已定案|
|`districts.py`|桃園市 13 個行政區，代碼如 `zhongli`|**暫定**，範圍還要跟大家確認|
|`tags.py`|10 個需求／專長標籤，如 `dementia_care`、`respect_communication`|**草稿**，只有 `respect_communication` 出自文件|

- **寫入前要驗證代碼。** `available_slots`、`service_districts`、`skill_tags`、`requested_slots`、`need_tags` 這些 JSON 欄位資料庫不檢查內容，寫入的 Service 要自己檢查代碼存在於字典。
- **性別代碼不一致。** `gender` 用 `male`／`female`，`Assessment.gender_pref` 用 `M`／`F`／`ANY`（文件 B-6 如此定義）。推薦模組比對時要自己轉換，建議之後統一。
- **標籤字典的擁有者還沒決定。** 題庫的選項要對應到 `tags.py` 的代碼，所以設計題庫的人必須參與定稿。

## 修改欄位與遷移流程

改資料表結構一律先改 `app/models/`，再產生遷移檔，不要手動改資料庫。**遷移檔只由一位維護者產生**（目前是成員 3），其他人改模型後通知維護者，這樣不會同時出現兩個遷移檔。

維護者的流程：

```bash
cd backend
alembic revision --autogenerate -m "說明這次改什麼"
# 打開 alembic/versions/ 裡新產生的檔，確認 upgrade() 的內容是你要的
alembic upgrade head
git add alembic/versions app/models
git commit -m "..."
```

其他人拉到新程式碼後，在 `backend/` 執行一次 `alembic upgrade head`，沒有資料要保留時也可以刪掉 `joycare.db` 重建。

- **新增模型檔後，要在 `app/models/__init__.py` 匯入。** 沒匯入的話 autogenerate 看不到這張表，會產生空的遷移。
- **模型沒變時，autogenerate 產生空遷移。** 這不是錯誤。
- **優先新增可空（`nullable=True`）的欄位。** SQLite 改欄位會走 batch 模式（重建表），新增可空欄位最安全。
- **同時出現兩個 head 時**（`alembic heads` 列出兩筆），表示有兩人各自產生了遷移，要由維護者合併。
- **開發初期尚未有人依賴資料時**，可以直接刪掉第一版遷移檔與 `joycare.db` 重新產生，不用一直疊遷移。

## Seed 帳號與資料

`python -m app.seed` 會寫入 8 個帳號、兩位被照顧者與 5 位照服員的資料，所有帳號的密碼都是 `password123`。

|角色|Email|
|---|---|
|個管師|`manager@joycare.test`|
|家屬|`family1@joycare.test`、`family2@joycare.test`|
|照服員|`caregiver1@joycare.test` 到 `caregiver5@joycare.test`|

- **Seed 只能在空資料庫執行。** 已有資料時會略過。如果上一次執行失敗、留下半套資料，再跑會因 email 重複失敗，要先重置資料庫（見「環境建置」）。
- **Seed 只在最後 commit 一次。** 中途失敗時整批回復，不會留下半套資料。修改 seed 時請維持這個結構。
- **只用於開發與測試。** 這些帳密不可用在任何對外部署的環境。
- **驗證密碼用 `verify_password`。** 資料庫存的是 argon2 雜湊字串，看不到明文，不能用字串比對來驗證。