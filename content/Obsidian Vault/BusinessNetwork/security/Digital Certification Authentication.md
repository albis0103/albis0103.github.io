#### 定義
- 由受信任憑證機構（CA, Certificate Authority）簽發，將public Key 與 身份綁定（數位簽章）
- 目前主流 X.509 v3
### X.509 憑證主要欄位

| 欄位                      | 說明                                                                                |
| ----------------------- | --------------------------------------------------------------------------------- |
| Version                 | v3（支援 extensions）                                                                 |
| Serial Number           | CA 內唯一，撤銷時用                                                                       |
| Signature Algorithm     | 如 sha256WithRSAEncryption、ecdsa-with-SHA384                                       |
| Issuer                  | 簽發者 Domain Number                                                                 |
| Validity                | 有效時間 ex. notBefore / notAfter                                                     |
| Subject                 | 主體 Domain Number                                                                  |
| Subject Public Key Info | 公鑰與演算法                                                                            |
| Extensions              | SAN、Basic Constraints、Key Usage、EKU、**AKI/SKI**、CRL Distribution Points、AIA、SCT 等 |
| Signature               | CA 對 TBSCertificate 的簽章                                                           |
關鍵 extensions：

- **Subject Alternative Name (SAN)**：瀏覽器以 SAN 比對主機名稱，CN 已不被採用（RFC 6125 / RFC 9110 慣例）。
- **Basic Constraints**：`CA:TRUE/FALSE` 與 `pathLenConstraint`，限制能否簽發下層憑證及鏈深度。
- **Key Usage / Extended Key Usage**：限定用途（digitalSignature、keyEncipherment、serverAuth、clientAuth、codeSigning）。
- **AIA**（Authority Information Access）：提供 OCSP responder 與簽發者憑證位置。
- **CRL Distribution Points**：CRL 下載位置。

- `Subject Key Identifier (SKI)`：本憑證**公鑰**的識別碼，公鑰的 SHA-1 雜湊
- `Authority Key Identifier (AKI)`：簽發者**公鑰**的識別碼，內容應等於簽發者憑證的 SKI。


#### PKI Infrastructure
[![What is PKI? Public Key Infrastructure Explained](https://rublon.com/wp-content/uploads/2026/02/pki-diagram2-scaled.png)


#### 憑證鏈流程
```
leaf (example.com)
  │ ① 以 Intermediate 公鑰驗證 leaf 簽章
		  current.IssuerID == candidate.subjectID;
		  current.AKI == candidate.SKI;
		  then 選出 candidate
  │ ② 檢查 leaf: 有效期、SAN、EKU=serverAuth、撤銷
  ▼
Intermediate CA
  │ ③ 以 Root 公鑰驗證 Intermediate 簽章
  │ ④ 檢查: CA:TRUE、pathLen、keyCertSign、Name Constraints、有效期、撤銷
  ▼
Root CA ← 在 Trust Store 中？是 → 建鏈與驗證成功
```


#### Challenge and Response 流程
![[Pasted image 20261021001754.png]]
1. Supplicant （被驗證者）宣稱自己是 True Party：被 Verifier 算出 challenge要求驗證
2. Supplicant 用 private key 簽章，處理 challenge message
3. Supplicant 回應 response message（封裝TLS handshake）
	- TLS handshake：`certificate`, `ClientKeyExchange`, `CertificateVerify`
	- `certificate_list : [0]: supplicant x.509, [1], .., [n]:  Immediate x.509 ` 
4. Verifier 事先或當場取得 CA 提供的 True party 憑證 （憑證鏈驗證）
5. 憑證包括 True Party 公鑰與公開資訊
6. Verifier 用 True party public key 還原response message 成 challenge，還原成功則驗證成功

#### Digital Certification Authentication 比 其他驗證方法安全
###### 核心差異：非對稱金鑰 + 不傳private key

|              | 密碼 / API Key / Bearer Token | 憑證驗證                                    |
| ------------ | --------------------------- | --------------------------------------- |
| 驗證依據         | 雙方共享的秘密                     | 簽章(私鑰只用來簽章)                             |
| 秘密是否離開持有者    | 是，每次驗證都傳送                   | 否，**私鑰不離開 client/server**               |
| Server 端儲存內容 | 密碼雜湊或 key 本體                | **僅公鑰/憑證（公開資料）**                        |
| 伺服器資料庫外洩影響   | 憑證資料可被離線破解或直接重放             | 外洩的是公開資料，無法用於冒充                         |
| 截獲通訊後能否重放    | Bearer token 可直接重放          | **簽章綁定 handshake transcript（TTL），不可重放** |

- Integrity：能確認檔案在途中是否被篡改
- Authentication：可以驗證 Sender的身份
- Non - Repudiation：Sender無法否認送出的 Response message