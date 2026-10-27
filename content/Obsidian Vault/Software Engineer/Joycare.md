

家屬填需求評估問卷，
系統產生需求標籤，
依規則計分推薦最多 3 位照服員。
家屬選人預約，
照服員接受或婉拒。
服務結束後家屬回饋，同類問題累積到 3 筆時，系統自動標記給個管師處理。

# 業務流程與各服務

系統有 6 個 Service，對應 6 個模組（C-7）。每張表只有一個 Service 能寫入。下面依 8 步流程，說明每步由誰處理、寫哪些表。

![[Pasted image 20261021143215.png]]

## 流程總覽

| 步驟  | 動作       | 負責 Service            | 寫入的表                                |
| --- | -------- | --------------------- | ----------------------------------- |
| 1   | 註冊建檔     | AccountService        | User、CareRecipient、CaregiverProfile |
| 2   | 需求評估精靈   | AssessmentService     | Assessment                          |
| 3   | 產生推薦     | RecommendationService | Recommendation                      |
| 4   | 家屬選人預約   | BookingService        | Booking（並改推薦狀態）                     |
| 5   | 照服員接受或婉拒 | BookingService        | Booking（並改推薦狀態）                     |
| 6   | 共享行事曆    | CalendarService       | ScheduleEvent                       |
| 7   | 回饋與上浮    | FeedbackService       | Feedback、`avg_rating`               |
| 8   | 個管師處理    | FeedbackService       | Feedback 的處理欄位                      |

## 逐步說明

### 1. 註冊建檔（AccountService，模組 1）

- 註冊時只能選家屬或照服員。個管師由 seed 建立。
- 家屬建立被照顧者，欄位有年齡、性別、行動能力、行政區、健康備註。
- 照服員維護自己的資料，欄位有年資、證照、服務區域、可服務時段、專長標籤、性別。
- 個管師可停用或啟用帳號（`is_active`），不能刪除。

### 2. 需求評估精靈（AssessmentService，模組 2）

- 家屬 `POST /assessments` 開始評估，再用 `POST /assessments/{id}/answers` 逐題作答。每次作答，後端回傳下一題。
- 下一題由 domain 的 `wizard.py` 決定。規則是先看 `next_question_rule.by_option` 有沒有對應答案的題號，沒有就走 `default`，為 null 表示結束。
- 選項帶的標籤累加成 `need_tags`。標籤是英文代碼，例如 `respect_communication`。
- 時段題和性別偏好題不轉成標籤。評估完成時，它們的答案寫入 `requested_slots` 與 `gender_pref`，之後作為推薦條件。
- 可以中途離開，回來從最後一題繼續。重新評估會建立新的 Ass
- essment。
- 個管師維護題庫：停用題目是軟刪除，儲存時檢查 `next_question_rule` 不能形成迴圈。

### 3. 產生推薦（RecommendationService，模組 3）

- 推薦在「評估完成」的同一個交易內自動產生，沒有獨立的觸發 API。
- 計分由 domain 的 `scoring.py` 完成，輸入是需求資料和照服員資料，輸出分數與推薦理由：
    - 專長 30（標籤交集比例）
    - 區域 25（行政區在服務區域內才給分）
    - 時段 25（時段交集比例）
    - 性別偏好 10（無偏好或符合才給分）
    - 評價 10（尚無評價以 3 分計）
- 排除條件：帳號已停用，或時段完全無交集。
- 取前 3 名，同分依 `avg_rating`、`years_exp`、`caregiver_id` 排序。
- 推薦結果是唯讀快照，之後只有 `status` 會變：`recommended` → `selected`／`not_selected`。

### 4. 家屬選人預約（BookingService，模組 4）

- 家屬 `POST /recommendations/{rec_id}/booking`，指定起訖日期與時段。預約只能由推薦建立，`recipient_id` 由推薦推導，不由前端傳入。
- 檢查：時段必須是該照服員可服務時段的子集。同一份評估同時只能有一筆 `pending` 或 `confirmed` 預約。
- 同一交易內：建立 `pending` 預約，並把該筆推薦改為 `selected`。

### 5. 照服員接受或婉拒（BookingService）

- **接受：** 檢查衝突。照服員自己已確認或進行中的預約，若日期重疊且時段有交集就拒絕（回 409 `BOOKING_CONFLICT`）。通過後預約變 `confirmed`，其餘推薦改 `not_selected`。檢查與寫入在同一交易內完成，SQLite 靠 WAL 加 `BEGIN IMMEDIATE` 防止併發超賣。
- **婉拒：** 預約變 `rejected`，該筆推薦改 `not_selected`，家屬可從同一份清單重選。
- **開始與結束：** 照服員 `start`（`confirmed` → `in_progress`），家屬或照服員 `end`（→ `ended`）。非法轉換由 domain 的 `booking_state.py` 擋下，回 409。

### 6. 共享行事曆（CalendarService，模組 5）

- 家屬和該預約的照服員都能新增、修改事件，類型有照護、就醫、用藥。個管師唯讀。
- 事件只能建在 `confirmed` 或 `in_progress` 的預約下，日期必須落在預約起訖之間。

### 7. 回饋與上浮（FeedbackService，模組 6）

- 預約 `ended` 後，家屬可填一則回饋（每筆預約限一則）：評分 1–5，問題類別（態度、衛生、技能，或無問題）和說明。
- 同一交易內完成三件事：建立回饋、全量重算該照服員的 `avg_rating`（只有這個 Service 能寫這個欄位）、計算「同照服員、同類別、尚未結案」的回饋數。達到 3 筆，這組回饋全部標記 `is_flagged`。
- 「無問題」的回饋不計入門檻。

### 8. 個管師處理（FeedbackService）

- 待辦清單就是已標記且尚未結案的回饋（`GET /feedbacks?flagged=true`）。
- 狀態依 `open` → `in_progress` → `closed` 推進，並記錄 `handled_by`。`is_flagged` 一旦設定就不清除。

## 三個 domain 純函式

這些不依賴資料庫，是單元測試的重點：

- `wizard.py`：決定下一題。
- `scoring.py`：推薦計分。
- `booking_state.py`：預約狀態機。

## 容易出問題的跨服務耦合

1. **Recommendation 的狀態由 BookingService 觸發。** Recommendation 的擁有者是 RecommendationService，但 BookingService 在預約動作時要改它的 `status`。文件規定 BookingService 呼叫 RecommendationService 的方法，並共用同一個 Session。
2. **`avg_rating` 的所有權例外。** 它在 CaregiverProfile，擁有者是 AccountService，但只允許 FeedbackService 寫。
3. **推薦輸入跨模組。** 計分用到 Assessment 的 `requested_slots`、`gender_pref` 和 CaregiverProfile 的 `gender`。這些是 Q1／AD-14 要補的欄位，沒補就算不出性別偏好。
4. **健康備註的讀取權限。** 照服員只有在預約為 `pending`、`confirmed`、`in_progress` 時才能讀，涉及 Booking 狀態與 CareRecipient 兩個模組。

![[Pasted image 20261021154023.png]]