
**定義**

對數位內容（文件、影像、影片、資料集等）完整生命週期建立詳細，不可串改的紀錄

**主要組成要素**

1. **來源（Origin）**：內容的初始建立者、建立工具、建立時間與環境
2. **鏈路（Chain of Custody）**：內容經過的每一次修改、轉移、處理的記錄，形成不可否認的歷程鏈
3. **完整性驗證（Integrity Verification）**：通常透過密碼學雜湊（hash）確保內容未被竄改
4. **可信簽章（Cryptographic Signing）**：以數位簽章綁定作者身份與內容版本

**技術實作方式**

- **中繼資料嵌入（Metadata Embedding）**：將溯源資訊直接嵌入檔案（如 C2PA 標準中的 manifest）
	- note: C2PA(Coalition for Content Provenance and Authentication)：不依賴集中資料庫，將Provenance 綁定檔案metadata，實現信任鏈（ex.相機內安全模組簽章，Photoshop再次簽章，but 截圖則無法留下）
- **雜湊鏈（Hash Chaining）**：每次修改都記錄前一版本的 hash，形成類似區塊鏈的鏈結結構
- **分散式帳本（Distributed Ledger / Blockchain）**：使用BlockChain儲存
- **簽章式溯源（Signed Provenance）**：如 C2PA（Coalition for Content Provenance and Authenticity）標準，由 Adobe、Microsoft、BBC 等主導，為影像/影片附加加密簽章的來源聲明（Content Credentials）

### process

### 1：Cryptographic Hashing


**實作方式**：

- 對整個檔案位元組序列計算雜湊值，作為內容指紋
- 用 **Merkle Tree** 結構：將檔案切分為區塊，每個區塊計算 leaf hash，逐層往上聚合成 root hash

```
Root Hash = H(H(Block1) || H(Block2) || ... || H(BlockN))
```

### 2.Digital Signature + PKI

**技術棧**：

- 非對稱加密
- 簽章對象：`Digital Envelop`(content_hash)
- 憑證鏈：X.509 憑證，綁定公鑰與身份，由受信任 CA（Certificate Authority）或內部 PKI 簽發

**流程**：

```
1. envelop = SHA256(content)
2. signature = Sign(private_key, envelop)
3. manifest = {envelop, signature, certificate_chain, timestamp}
```

**驗證方**：

```
1. 重新計算 content_hash' = SHA256(received_content)
2. 驗證 content_hash' == envelop（完整性）
3. Verify(public_key, signature, content_hash)（真實性）
4. 驗證憑證鏈至受信任 root CA（身份可信度）
```

---

## C2PA 標準

C2PA（由 Adobe、Microsoft、Truepic、BBC 等成立）是目前最成熟的可互通溯源標準，核心結構如下：

### 1. Manifest 結構

一組嵌入檔案內的 JSON/CBOR 結構化資料，包含：

- **Assertions**：對內容的陳述，例如：
    - `c2pa.hash.data`：內容雜湊
    - `c2pa.actions`：動作歷程（如 `c2pa.created`, `c2pa.edited`, `c2pa.opened`）
    - `c2pa.ai.generative`：標示是否為 AI 生成
    - `stds.exif`：EXIF 中繼資料
- **Claim**：由建立者對這些 assertions 做的簽名聲明，包含 assertions 的雜湊清單
- **Claim Signature**：對 Claim 的密碼學簽章（COSE 格式，Ed25519 或 ES256）
- **Ingredients**：若內容由多個來源合成（例如 AI 編輯過的照片），記錄每個輸入來源的 manifest（形成 **溯源鏈的遞迴結構**）

### 2 嵌入方式

- **JPEG/PNG**：嵌入於檔案的自訂 metadata segment（如 JPEG 的 APP11 marker）
- **MP4/影片**：嵌入於容器格式的 metadata box
- **雲端斷開時的容錯**：若 metadata 被剝離（例如上傳到不支援的平台），可透過 **soft binding**（如視覺浮水印或感知雜湊 fallback）或 C2PA 的 **cloud manifest repository** 查詢

### 2.3 實作工具

- `c2pa-rs`（Rust 官方 SDK，有 Python binding `c2pa-python`）
- 基本操作範例（Python）：

```python
from c2pa import Builder, Reader

# 建立 manifest 並簽署
builder = Builder({
    "claim_generator": "MyApp/1.0",
    "assertions": [
        {"label": "c2pa.actions", "data": {"actions": [{"action": "c2pa.created"}]}}
    ]
})
builder.sign(signer_cert, signer_key, source_path, output_path)

# 讀取並驗證
reader = Reader(output_path)
manifest = reader.get_active_manifest()
```

---

## 3. Hashing與分散式帳本層



### 1 Hash Chain

每次修改記錄前一版本的 hash，形成鏈式結構：

```
Version_N.prev_hash = H(Version_{N-1})

Version_N.signature = Sign(
	editor_key, Version_N.content_hash || Version_N.prev_hash
)
```


### 2 Blockchain-based Provenance

僅在需要**去中心化信任**（不依賴單一 CA 或中心化資料庫）時才需要：

- 將內容 hash（而非內容本身）寫入區塊鏈交易（如 Ethereum、Hyperledger Fabric）
- 用途：取得不可竄改的**時間戳證明**（timestamping），證明「某內容在某時間點已存在且未變」
- 常見誤用：不需要為了溯源而把整個檔案上鏈，只需上鏈 hash + metadata pointer（IPFS CID 等）

實務上，多數企業場景用 **Certificate Transparency 式的 append-only log**（如 Trillian）即可達到類似效果，不必動用區塊鏈。

---

## 4. 軟體供應鏈場景：SBOM + Provenance Attestation

若情境是軟體/CI-CD pipeline，對應標準是 **SLSA（Supply-chain Levels for Software Artifacts）** + **in-toto**：

### 4.1 in-toto Attestation

記錄軟體建置流程每個步驟（source → build → test → deploy）的 attestation：

```json
{
  "predicateType": "https://slsa.dev/provenance/v1",
  "subject": [{"name": "artifact.tar.gz", "digest": {"sha256": "..."}}],
  "predicate": {
    "buildDefinition": {"buildType": "...", "externalParameters": {...}},
    "runDetails": {"builder": {"id": "..."}, "metadata": {"invocationId": "..."}}
  }
}
```

### 4.2 實作工具鏈

- **Sigstore**（`cosign` + `rekor` + `fulcio`）：業界目前最實用的落地方案
    - `cosign sign`：對容器映像/artifact 簽章
    - `fulcio`：短期憑證頒發（keyless signing，綁定 OIDC 身份而非長期私鑰）
    - `rekor`：transparency log，記錄所有簽章事件，公開可驗證且不可竄改
- **SBOM 格式**：SPDX 或 CycloneDX，記錄軟體組件清單，可與 attestation 綁定

```bash
# 對 artifact 簽章並寫入 transparency log
cosign sign --key cosign.key artifact.tar.gz

# 驗證
cosign verify --key cosign.pub artifact.tar.gz
```
