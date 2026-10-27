將多個 `SELECT` 結果 垂直合併，會刪除重複紀錄
#### Prerequisite
- 查詢欄位數要相同
- DataType要一樣，對應位置需相容或能做隱氏轉換

##### Example
```sql
SELECT name, price
FROM product
WHERE category = '3c'
UNION
SELECT username, password
FROM users;
```
- output

| name   | price      |
| ------ | ---------- |
| iPhone | 30000      |
| Mac    | 50000      |
| admin  | hash_12345 |
| user1  | hash_abcde |
