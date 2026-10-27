是一個將一次攻擊拆成連續階段的模型，常用於定位目前攻擊並決策阻斷。**（真武帝立案執行）**


| Stage                            | 攻擊                                                  | 防禦方法             |
| -------------------------------- | --------------------------------------------------- | ---------------- |
| Reconnaissance<br>（偵查）           | 搜集目標資訊 (Domain, IP, 員工..)<br>ex. ONIST, port scan   | 監控異常掃描           |
| Weaponization<br>（武器化）           | 將惡意程式與payload組合成可遞送物件<br>ex. Office Macro, 植入 PDF.. | 威脅情資追蹤           |
| Delivery<br>（遞送）                 | 將武器送達目標<br>ex. Phishing, USB, 惡意連結                  | 郵件Gateway, URL過濾 |
| Exploitation<br>（利用）             | 利用漏洞誘使 User 執行<br>ex. 利用CVE,                        | 修補管理, EDR        |
| Installation<br>（安裝）             | 植入持久化後門<br>ex. 排程任務, webshell                       | EDR, 最小權限        |
| C2, Command & Control<br>（指揮與控制） | 與受害主機建立通道<br>ex. HTTPS beacon, DNS tunneling        | 出站流量過濾           |
| Action<br>行動                     | 執行最終目的<br>ex. 橫向移動, 提權, 資料外洩                        | 網段隔離、行爲偵測        |

