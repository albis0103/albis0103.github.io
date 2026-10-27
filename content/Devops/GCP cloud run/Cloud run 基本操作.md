

## 1. 核心概念

- **Service**：長駐的 HTTP/gRPC 服務，每次部署產生一個不可變的 **Revision**，流量可在 revision 間切分。
- **Job**：執行到完成的批次任務（無 HTTP 端點），可平行多個 task，可由 Cloud Scheduler 觸發。
- **容器契約**：監聽 `0.0.0.0:$PORT`（預設 8080）、無狀態、啟動需在逾時內完成 listen。
- **計費**：預設為 request-based（僅請求處理期間計 CPU）；可選 instance-based（CPU always allocated）。
- **並行度（concurrency）**：單一 instance 同時處理的請求數，預設 80，最大 1000。

## 2. 前置設定

```bash
gcloud auth login
gcloud config set project PROJECT_ID
gcloud config set run/region asia-east1   # 台灣彰化

gcloud services enable run.googleapis.com \
  artifactregistry.googleapis.com cloudbuild.googleapis.com
```

## 3. 部署方式

### 3.1 從原始碼直接部署（Buildpacks / Dockerfile）

```bash
gcloud run deploy my-service \
  --source . \
  --region asia-east1 \
  --allow-unauthenticated
```

會自動透過 Cloud Build 建置並推送至 Artifact Registry。若目錄有 `Dockerfile` 則使用之，否則使用 Buildpacks。

### 3.2 從既有映像部署

```bash
# 建立 repository
gcloud artifacts repositories create my-repo \
  --repository-format=docker --location=asia-east1

# 建置並推送
gcloud builds submit --tag \
  asia-east1-docker.pkg.dev/PROJECT_ID/my-repo/my-service:v1

# 部署
gcloud run deploy my-service \
  --image asia-east1-docker.pkg.dev/PROJECT_ID/my-repo/my-service:v1 \
  --region asia-east1
```

### 3.3 最小 Dockerfile 範例（Python / FastAPI）

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD exec uvicorn main:app --host 0.0.0.0 --port ${PORT:-8080}
```

## 4. 常用部署參數

|參數|說明|
|---|---|
|`--cpu` / `--memory`|例如 `--cpu 2 --memory 1Gi`|
|`--min-instances` / `--max-instances`|控制冷啟動與成本上限|
|`--concurrency`|單 instance 並行請求數|
|`--timeout`|請求逾時（最大 3600s）|
|`--set-env-vars KEY=VAL`|環境變數|
|`--set-secrets KEY=SECRET:latest`|從 Secret Manager 注入|
|`--service-account`|執行身分|
|`--no-cpu-throttling`|instance-based 計費，CPU 常駐|
|`--ingress`|`all` / `internal` / `internal-and-cloud-load-balancing`|
|`--vpc-connector` 或 `--network`/`--subnet`|存取 VPC 內資源（Direct VPC egress）|

## 5. 存取控制（IAM）

```bash
# 公開存取
gcloud run services add-iam-policy-binding my-service \
  --member=allUsers --role=roles/run.invoker --region asia-east1

# 限定特定 service account 呼叫
gcloud run services add-iam-policy-binding my-service \
  --member=serviceAccount:caller@PROJECT_ID.iam.gserviceaccount.com \
  --role=roles/run.invoker --region asia-east1

# 呼叫需驗證的服務
curl -H "Authorization: Bearer $(gcloud auth print-identity-token)" SERVICE_URL
```

預設要求驗證（`--no-allow-unauthenticated`）；服務間呼叫使用 ID token，audience 為 service URL。

## 6. Revision 與流量管理

```bash
gcloud run revisions list --service my-service

# 部署新版但不導流（canary 前置）
gcloud run deploy my-service --image ...:v2 --no-traffic --tag canary

# 切分流量
gcloud run services update-traffic my-service \
  --to-revisions my-service-00002-abc=10,my-service-00001-xyz=90

# 全部導向最新版 / 回滾
gcloud run services update-traffic my-service --to-latest
gcloud run services update-traffic my-service --to-revisions REV_NAME=100
```

`--tag canary` 會產生專屬 URL（`https://canary---my-service-xxx.run.app`）供驗證。

## 7. 設定管理

```bash
# 更新環境變數（不重建映像，會產生新 revision）
gcloud run services update my-service --update-env-vars LOG_LEVEL=debug

# Secret Manager
echo -n "value" | gcloud secrets create DB_PASSWORD --data-file=-
gcloud run services update my-service --set-secrets DB_PASSWORD=DB_PASSWORD:latest
```

Cloud Run 的 service account 需有 `roles/secretmanager.secretAccessor`。

## 8. 觀測與除錯

```bash
gcloud run services describe my-service          # 狀態、URL、設定
gcloud run services list
gcloud run services logs read my-service --limit 50
gcloud logging read 'resource.type="cloud_run_revision" AND resource.labels.service_name="my-service"' --limit 20
```

stdout/stderr 自動進入 Cloud Logging；指標（request count、latency、instance count）在 Cloud Monitoring。

常見啟動失敗原因：未監聽 `$PORT`、監聽 `127.0.0.1` 而非 `0.0.0.0`、啟動逾時、記憶體不足（OOM）、service account 權限不足。

## 9. Cloud Run Jobs

```bash
gcloud run jobs create my-job \
  --image ...:v1 --tasks 5 --max-retries 2 --task-timeout 600s

gcloud run jobs execute my-job --wait
gcloud run jobs executions list --job my-job
```

task 內可用環境變數 `CLOUD_RUN_TASK_INDEX` 與 `CLOUD_RUN_TASK_COUNT` 分片處理資料。

## 10. 自訂網域與刪除

```bash
gcloud beta run domain-mappings create --service my-service --domain api.example.com
gcloud run services delete my-service
```

生產環境建議改用外部 Application Load Balancer + serverless NEG 以取得 Cloud Armor、CDN 與多區域能力。

## 11. 注意事項

- **檔案系統**為記憶體內暫存，instance 結束即消失；持久化請用 Cloud Storage、Cloud SQL、Firestore，或掛載 Cloud Storage FUSE / NFS volume。
- **冷啟動**：設 `--min-instances 1` 可避免，但會產生閒置成本；亦可啟用 startup CPU boost。
- **背景執行緒**在 request-based 計費下，請求結束後 CPU 受限制；需背景工作請用 `--no-cpu-throttling`。
- 單請求逾時上限 60 分鐘，長時間任務改用 Jobs 或 Cloud Tasks。

---

以上為依我既有知識整理，部分旗標（如 Direct VPC egress 相關參數）可能隨版本變動，以 `gcloud run deploy --help` 或官方文件為準。若需要，我可以針對特定情境（FastAPI + Cloud SQL、與 Cloud Scheduler 串接、CI/CD via GitHub Actions）給出完整範例。