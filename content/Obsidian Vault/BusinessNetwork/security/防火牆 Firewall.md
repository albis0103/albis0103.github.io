部署在網路邊界或主機上的存取控制機制，依據預先定義規則檢查經過的封包或連線，決定 allow , drop 或 reject
- Ingress Filtering：過濾進站封包
- Egress Filtering：過濾出站封包，防止資料外洩

### 運作層級與類型

| 類型                        | 作用層 (OSI)      | 判斷依據                                        | 特性                          |
| ------------------------- | -------------- | ------------------------------------------- | --------------------------- |
| 封包過濾（Stateless）           | L3/L4          | 來源/目的 IP、Port、協定                            | 逐封包獨立判斷，無連線狀態，效能高但規則需雙向明確定義 |
| 狀態檢測（Stateful Inspection） | L3/L4          | 上述欄位 + 連線狀態表（如 TCP NEW/ESTABLISHED/RELATED） | 回程流量自動對應已建立連線，現行主流基礎能力      |
| 應用層閘道／代理（Proxy）           | L7             | 應用協定內容                                      | 終止並重建連線，可深度檢查，延遲與資源成本較高     |
| 次世代防火牆（NGFW）              | L3–L7          | 應用識別（App-ID）、使用者身分、IPS 特徵、TLS 檢查、威脅情資       | 整合 DPI、IPS、URL 過濾、沙箱等       |
| WAF                       | L7（HTTP/HTTPS） | HTTP 請求內容（SQLi、XSS 等特徵）                     | 專門保護 Web 應用，與網路防火牆互補而非取代    |

### 狀態檢測的核心機制

連線狀態表（conntrack table）以五元組（src IP, dst IP, src port, dst port, protocol）為 key，記錄 TCP 狀態（SYN_SENT → ESTABLISHED → FIN_WAIT…）與逾時。封包到達時：

1. 查狀態表，若屬已建立連線，直接放行。
2. 否則依規則由上而下比對（first-match），命中即執行動作。
3. 無任何規則命中時套用預設策略（default policy）。

- SPI: 對建立連線封包比對 ACL (規則)，其餘查表放行，缺點是**只看 Socket**
- NGFW：深度檢查 + 應用識別 + IDP / IPS 