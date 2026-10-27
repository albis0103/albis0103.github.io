


## 1. 建立資料庫

```sql
CREATE DATABASE library_db
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;
```
`CHARACTER SET` 與 `COLLATE` 是可選子句，設定的是這個資料庫的預設值，之後建表時如果沒有另外指定，就會沿用。

**字元集（CHARACTER SET）**：定義字元與位元組之間的編碼方式。

|字元集|每字元最大長度|說明|
|---|---|---|
|`utf8`（= `utf8mb3`）|3 bytes|MySQL 的舊實作，存不了 emoji 和部分罕用字|
|`utf8mb4`|4 bytes|完整的 UTF-8，現行標準|

**定序（COLLATE）**：定義字串的比較與排序規則，會影響 `=`、`ORDER BY`、`GROUP BY` 和 UNIQUE 判定。命名格式是 `字元集_規則_屬性`：

- `ci`：不分大小寫，`'abc' = 'ABC'` 的結果是 1
- `cs`：區分大小寫
- `bin`：逐位元組比較
- `ai`：不分重音符號
- `unicode`：依 UCA 4.0 規則；`0900` 則依 UCA 9.0，較新也較快

**各版本的預設值**：MySQL 5.7 是 `latin1`（存中文會亂碼），MySQL 8.0 是 `utf8mb4` / `utf8mb4_0900_ai_ci`。因為預設值隨版本不同，建議明確指定。

```sql
SHOW CREATE DATABASE library_db;   -- 確認設定
```

## 2. 建立使用者

```sql
CREATE USER 'library_user'@'localhost' IDENTIFIED BY '密碼';
```

- MySQL 的帳號由 `'使用者'@'主機'`定義。
    - `localhost`：只允許從本機連線
    - `%`：允許從任何主機連線
- `IDENTIFIED BY` 設定密碼，雜湊後存在 `mysql.user` 系統表。
- MySQL 8.0 的預設驗證插件是 `caching_sha2_password`，這和 `allowPublicKeyRetrieval` 有關。

## 3. 授予權限

```sql
GRANT 權限 ON 範圍 TO '使用者'@'主機';
```

|範圍|意義|
|---|---|
|`*.*`|全伺服器（等同 root 等級）|
|`library_db.*`|指定資料庫的所有表|
|`library_db.books`|單一資料表|

```sql
-- 全部權限（ALL PRIVILEGES 可簡寫為 ALL）
GRANT ALL PRIVILEGES ON library_db.* TO 'library_user'@'localhost';

-- 最小權限：只有 CRUD
GRANT SELECT, INSERT, UPDATE, DELETE ON library_db.* TO 'library_user'@'localhost';

-- 確認權限
SHOW GRANTS FOR 'library_user'@'localhost';
```

權限範圍要配合 Hibernate 的 `ddl-auto` 設定：`update` 和 `create` 需要 `CREATE`、`ALTER`、`DROP` 等 DDL 權限，只給 CRUD 的話，應用程式啟動時會失敗。

## 4. FLUSH PRIVILEGES

這個指令會從系統表重新載入權限到記憶體。

- 用 `CREATE USER`、`GRANT`、`REVOKE` 修改時，變更會立即生效，**不需要** FLUSH。
- 只有直接用 `INSERT` 或 `UPDATE` 修改 `mysql.user` 等系統表時，才需要 FLUSH。

## 5. Spring Boot 連線設定（application.properties）

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/library_db?allowPublicKeyRetrieval=true&useSSL=false&serverTimezone=Asia/Taipei
spring.datasource.username=library_user
spring.datasource.password=密碼

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

### 5.1 JDBC URL

|片段|意義|
|---|---|
|`jdbc:mysql://`|使用 MySQL 的 JDBC driver（Connector/J）|
|`localhost:3306`|主機與埠號，3306 是 MySQL 的預設埠|
|`/library_db`|連線後預設使用的資料庫|
|`?a=1&b=2`|連線參數，用 `&` 分隔|

|參數|作用|適用範圍|
|---|---|---|
|`useSSL=false`|關閉 TLS 加密，連線以明文傳輸|僅限本機開發；正式環境用 `sslMode=REQUIRED`（新版 driver 以 `sslMode` 取代 `useSSL`）|
|`allowPublicKeyRetrieval=true`|在沒有 TLS 時，允許 client 向 server 取得 RSA 公鑰來加密密碼。沒設定會報錯 `Public Key Retrieval is not allowed`|僅限本機開發；有中間人偽造公鑰的風險|
|`serverTimezone=Asia/Taipei`|指定伺服器時區，避免 Java 時間型別和 `DATETIME`/`TIMESTAMP` 轉換時偏移|通用|

### 5.2 帳號密碼

`username` 和 `password` 就是第 2 節建立的帳號。帳號的 host 是 `'localhost'`，所以 URL 也必須連 `localhost`。

### 5.3 JPA / Hibernate

`ddl-auto` 控制 Hibernate 啟動時是否根據 `@Entity` 自動修改 schema：

|值|行為|適用|
|---|---|---|
|`none`|不做任何事|正式環境|
|`validate`|只檢查 Entity 和資料表是否一致，不一致就啟動失敗|正式環境|
|`update`|自動新增缺少的表或欄位；不刪除多餘欄位，也不修改既有欄位的型別|開發初期|
|`create`|啟動時刪除後重建，資料會清空|測試|
|`create-drop`|同 `create`，並在關閉時再刪除|單元測試|

正式環境的 schema 通常改由 Flyway 或 Liquibase 管理。

- `show-sql=true`：把 Hibernate 產生的 SQL 印到 stdout（不經過 logging 框架）。
- `format_sql=true`：把印出的 SQL 排版成多行縮排。
- `spring.jpa.properties.*`：這個前綴底下的設定會原樣傳給 Hibernate。

## 6. 錯誤排查紀錄

|錯誤|原因|修正|
|---|---|---|
|`ERROR 3619: Illegal privilege level specified for ALL_PRIVILEGES`|寫成 `ALL_PRIVILEGES`（有底線）。MySQL 8.0 把帶底線的名稱當成動態權限，而動態權限只能授予在 `*.*`|改成 `ALL PRIVILEGES`（空格）或 `ALL`|
|`ERROR 1064` 語法錯誤|關鍵字拼錯（`GRATE`）|改成 `GRANT`；錯誤訊息的 `near '...'` 會指出出錯的位置|
|`ERROR 1410`|GRANT 的對象帳號不存在（例如 host 拼成 `licalhost`）。MySQL 8.0 的 GRANT 不會自動建立帳號|先 `CREATE USER`，並確認 host 拼字|
|GRANT 成功但看不到資料庫|資料庫名稱不一致（`libraryDB` vs `library_db`）。Linux 上名稱預設區分大小寫，而 GRANT 指定不存在的資料庫**不會報錯**|先用 `SHOW DATABASES;` 確認實際名稱|
|`Public Key Retrieval is not allowed`|`caching_sha2_password` 搭配 `useSSL=false`，driver 預設禁止取得公鑰|URL 加上 `allowPublicKeyRetrieval=true`|
|存進去的時間差 8 小時|JVM 和 MySQL 的時區不一致|URL 加上 `serverTimezone=Asia/Taipei`|

## 7. 完整流程

```sql
-- MySQL（用 root 登入後執行）
CREATE DATABASE library_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'library_user'@'localhost' IDENTIFIED BY '密碼';
GRANT ALL PRIVILEGES ON library_db.* TO 'library_user'@'localhost';
SHOW GRANTS FOR 'library_user'@'localhost';
```

```bash
# 驗證帳號能登入
mysql -u library_user -p library_db
```

驗證成功後，再把帳號密碼填入 `application.properties`，然後啟動 Spring Boot。

需要的話，我可以把這份整理成 .md 檔給你下載。