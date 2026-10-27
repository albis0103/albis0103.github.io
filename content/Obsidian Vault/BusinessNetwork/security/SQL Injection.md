

#### Union-Based  
- 攻擊者透過 `UNION`把要查詢的隱私資訊（`username, password`）合併到原結果後面[[SQL Union]]
- 由於`UNION`要求欄位數一樣，要先用`ORDER BY`猜欄位數
	- input `1 ORDER BY 1--`：正常
	- input `1 ORDER BY 4--`：錯誤，表示查詢無4欄
	- 持續試
```sql
SELECT name, price
FROM products
WHERE category = '3c'

UNION

SELECT
username, password
FROM users;
```

##### Boolean-Based Blind  
透過**後端判斷查詢不存在或錯誤的微小頁面差異**，判斷一一拼湊目標資訊
- ex. web 後端
```python
user = db.query("SELECT * FROM users WHERE ...")

if user :
	# 比對密碼，失敗
	return "wrong pwd"
else:
	return "account not existed"
```
- 第一個字是'b': True，畫面: `wrong pwd`
- 第一個字是'a': False，畫面: `account not existed`
- 反復嘗試

```sql
AND SUBSTRING((SELECT current_user), 1, 1) = 'a'
```

#### Subverting Application Logic  
經典方式，改變SQL查詢結構繞過驗證邏輯
- ex. 輸入 `admin'--`

```sql
SELECT *
FROM users
WHERE username = 'admin' -- AND password = '--'
```

`--` 把密碼檢查註解掉;或用 `' OR '1'='1` 讓條件恆真,也常用於越權存取他人資料。

#### SQLMap  
自動化注入偵測/利用工具,涵蓋 Union、Boolean blind、Error-based、Time-based blind、Stacked queries,能自動判斷 DBMS、列舉資料庫/表/欄位並 dump 資料,特定情況下可達成 OS 指令執行。

```bash
sqlmap -u "http://target.com/page.php?id=1" \
	--dbs
sqlmap -u "http://target.com/page.php?id=1" \
	-D dbname \
	--tables
sqlmap -u "http://target.com/page.php?id=1" \
	-D dbname \
	-T users \
	--dump
```

#### Parameterized Query(防禦)  
SQL 語法結構與使用者輸入資料分開傳給資料庫,輸入永遠只當資料、不會被當語法執行。比單純跳脫特殊字元(escaping)更可靠,escaping 易被編碼或二階注入繞過。侷限:表名/欄位名無法參數化,需用白名單處理。

```java
String username = request.getParameter("username");
String password = request.getParameter("password");

String sql = "SELECT * FROM users WHERE username = ? AND password = ?";

PreparedStatement ps = conn.prepareStatement(query);
ps.setString(1, username);
ps.setString(2, password);

ResultSet rs = ps.executeQuery();
```

#### ORM 
ORM會強制參數化查詢 (Parameterized Query)
- ex. python sqlachemy
```python
username = "admin ' OR '1' = '1"
query = select(User).where(User.username == username)
session.execute(query).scalars().all()
```
- `username`是SQL 語法樹的 leaf
- 