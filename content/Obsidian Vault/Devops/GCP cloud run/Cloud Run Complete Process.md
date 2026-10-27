
![[Pasted image 20261013233524.png]]

Client
1. DNS (AnyCast IP)
2. cloud Run Front-end (Reverse Proxy, TLS terminate)：Ingress 設定 + IAM $\rightarrow$ Cloud Run [[Cloud Run Front-end]]
	- Ingress check：檢查IP
	- IAM (`run.invoker`) 驗證：檢查權限
	- reversion/ tag Traffic Split(依 traffic split)：版本分流
3. Request routing layer：選 instance （autoscaler 推薦 instance 優先）底層是 [[Knative 架構]]，$n$ concurrency的 service有 $n$ 個 slot
	- 有 slot: instance sandbox ([[gVisor]] or gen2) $\rightarrow$ container(image 載入 → 容器啟動) $\rightarrow$ response(startup probe)
	- 無 slot: pending queue: $\rightarrow$ 
		- Metrics-based autoscaler： on-demand scaling $\rightarrow$ new instance
4. Direct VPC egress / connector $\rightarrow$ Cloud NAT / public IP
	- Server less VPC Connector：GCP會在VPC建 Bridge VMs把流向轉出到VPC資源 (ex. Cloud SQL)
	- Direct VPC egress: Cloud Run instance 直接掛在你指定的 subnet，從 subnet 取得 IP
## ① 入口層

|路徑|說明|
|---|---|
|預設|`*.run.app` 或 domain mapping，經 GFE 終止 TLS|
|自建 ALB|Global External ALB → Serverless NEG，可掛 Cloud Armor / CDN / IAP|

```bash
# 預設路徑：自訂網域（beta）
gcloud beta run domain-mappings create --service my-service --domain api.example.com

# ALB 路徑：建立 Serverless NEG 並掛到 backend service
gcloud compute network-endpoint-groups create my-neg \
  --region=asia-east1 --network-endpoint-type=serverless \
  --cloud-run-service=my-service

gcloud compute backend-services create my-backend \
  --global --load-balancing-scheme=EXTERNAL_MANAGED

gcloud compute backend-services add-backend my-backend --global \
  --network-endpoint-group=my-neg --network-endpoint-group-region=asia-east1
# 之後還需建立 url-map、target-https-proxy、forwarding-rule、憑證
```

## ② 驗證層（Router）

在請求進入 instance 之前執行，被拒絕的請求不觸發擴容。

```bash
# ingress 限制：all | internal | internal-and-cloud-load-balancing
gcloud run services update my-service --ingress internal-and-cloud-load-balancing

# IAM：公開 / 限定 service account
gcloud run services add-iam-policy-binding my-service --region asia-east1 \
  --member=allUsers --role=roles/run.invoker
gcloud run services add-iam-policy-binding my-service --region asia-east1 \
  --member=serviceAccount:caller@PROJECT_ID.iam.gserviceaccount.com \
  --role=roles/run.invoker

# 呼叫需驗證的服務（ID token，audience = service URL）
curl -H "Authorization: Bearer $(gcloud auth print-identity-token)" SERVICE_URL
```

ALB 前置時，通常搭配 `--ingress internal-and-cloud-load-balancing`，避免繞過 ALB 直接打 `run.app`（Cloud Armor 會被繞過）。

## ③ 路由層（Revision 與流量）

每次 deploy 或 update 產生不可變的 Revision，Router 依 traffic split 分派。

```bash
gcloud run revisions list --service my-service

# 部署新版但不導流，並給專屬 URL 驗證
gcloud run deploy my-service --image ...:v2 --no-traffic --tag canary

# Canary 切分
gcloud run services update-traffic my-service \
  --to-revisions my-service-00002-abc=10,my-service-00001-xyz=90

# 全量切換 / 回滾
gcloud run services update-traffic my-service --to-latest
gcloud run services update-traffic my-service --to-revisions REV_NAME=100
```

## ④ 排程與擴縮層

行為：有可用 concurrency slot 就直接派送；否則請求排隊並觸發擴容，擴容受 `max-instances` 與配額限制，佇列逾時回 429 / 503。擴容依據是 concurrency 與 CPU 使用率（約 60% 目標），不是等到滿載才擴。

```bash
gcloud run services update my-service \
  --concurrency 80 \
  --min-instances 0 \
  --max-instances 10 \
  --cpu 1 --memory 512Mi \
  --timeout 300
```

|參數|影響|
|---|---|
|`--concurrency`|單 instance 同時請求數，越低越早擴容|
|`--min-instances`|≥1 可避免冷啟動，但有閒置成本|
|`--max-instances`|擴容上限，同時是成本與下游保護的上限|
|`--timeout`|單請求逾時（最大 3600s），也是佇列等待上限|

## ⑤ 實例層（冷啟動與 CPU 模式）

冷啟動順序：分配 sandbox → 從內部儲存載入 image → 啟動容器 → startup probe 通過 → 接流量。Image 在部署時已複製到 Cloud Run 內部，不是每次從 Artifact Registry 拉取。

```bash
# 建置映像（Cloud Build → Artifact Registry）
gcloud artifacts repositories create my-repo --repository-format=docker --location=asia-east1
gcloud builds submit --tag asia-east1-docker.pkg.dev/PROJECT_ID/my-repo/my-service:v1

# 部署（或 --source . 一步完成建置與部署）
gcloud run deploy my-service \
  --image asia-east1-docker.pkg.dev/PROJECT_ID/my-repo/my-service:v1 \
  --region asia-east1

# 冷啟動與 CPU 模式
gcloud run services update my-service --cpu-boost          # 啟動期加配 CPU
gcloud run services update my-service --no-cpu-throttling  # instance-based，CPU 常駐
```

自訂 startup probe（HTTP path、閾值）建議用 YAML 搭配 `gcloud run services replace service.yaml` 設定。

容器契約：監聽 `0.0.0.0:$PORT`、無狀態；收到 SIGTERM 後約 10 秒清理，之後 SIGKILL。

## ⑥ 執行層（設定注入）

```bash
# 環境變數（產生新 revision）
gcloud run services update my-service --update-env-vars LOG_LEVEL=debug

# Secret Manager（執行身分需 roles/secretmanager.secretAccessor）
echo -n "value" | gcloud secrets create DB_PASSWORD --data-file=-
gcloud run services update my-service --set-secrets DB_PASSWORD=DB_PASSWORD:latest

# 執行身分
gcloud run services update my-service \
  --service-account app-sa@PROJECT_ID.iam.gserviceaccount.com
```

檔案系統為記憶體內暫存，持久化需用 Cloud Storage、Cloud SQL、Firestore，或掛載 GCS FUSE / NFS volume。

## ⑦ 出站層

|需求|設定|
|---|---|
|一般公網，IP 不固定|不需設定|
|存取 VPC 私有資源|Direct VPC egress 或 connector，`--vpc-egress=private-ranges-only`|
|固定出口 IP|`--vpc-egress=all-traffic` + Cloud NAT（靜態 IP）|
|Google API|Private Google Access / PSC|

```bash
# Direct VPC egress
gcloud run services update my-service \
  --network=my-vpc --subnet=my-subnet \
  --vpc-egress=private-ranges-only

# Serverless VPC Access connector
gcloud run services update my-service \
  --vpc-connector=my-connector --vpc-egress=all-traffic
```

## ⑧ 觀測層

```bash
gcloud run services describe my-service     # URL、revision、設定
gcloud run services list
gcloud run services logs read my-service --limit 50
gcloud logging read 'resource.type="cloud_run_revision" AND resource.labels.service_name="my-service"' --limit 20
```

啟動失敗的常見原因對應到上述層級：

|症狀|層級|原因|
|---|---|---|
|502 / 部署失敗|⑤|未監聽 `$PORT`、綁在 `127.0.0.1`、啟動逾時、OOM|
|403|②|缺 `run.invoker`，或 ingress 限制擋下|
|429 / 503|④|達 `max-instances` 或配額，佇列逾時|
|讀不到 secret|⑥|service account 缺權限|
|連不到內網資源|⑦|未設 VPC egress，或 egress 模式不符|

## 補充：Jobs（另一條執行路徑）

Jobs 沒有 ② 到 ④ 的請求路徑，由 `execute` 或 Cloud Scheduler 觸發，task 跑完即結束。

```bash
gcloud run jobs create my-job --image ...:v1 --tasks 5 --max-retries 2 --task-timeout 600s
gcloud run jobs execute my-job --wait
gcloud run jobs executions list --job my-job
```

Task 內以 `CLOUD_RUN_TASK_INDEX` 與 `CLOUD_RUN_TASK_COUNT` 做資料分片。

Direct VPC egress、startup probe 旗標與 autoscaler 閾值可能隨 gcloud 版本調整，以 `gcloud run deploy --help` 與官方文件為準。