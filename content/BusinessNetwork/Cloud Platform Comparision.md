

## 一、概覽

|項目|AWS (Amazon Web Services)|Microsoft Azure|Google Cloud Platform (GCP)|
|---|---|---|---|
|推出年份|2006|2010|2008（App Engine）/ 2011（GCE GA 前後）|
|市佔（約略）|約 30% 以上，居首|約 20% 以上，居次|約 10% 以上，居三|
|優勢定位|服務最完整、生態最成熟|企業整合（AD、M365、Windows Server）|資料分析、ML、Kubernetes、網路|
|全球架構|Region / Availability Zone|Region / Availability Zone|Region / Zone|

## 二、核心服務對照

|類別|AWS|Azure|GCP|
|---|---|---|---|
|虛擬機|EC2|Virtual Machines|Compute Engine|
|容器編排|EKS|AKS|GKE|
|無伺服器容器|ECS Fargate / App Runner|Container Apps|Cloud Run|
|函式運算|Lambda|Functions|Cloud Functions / Cloud Run functions|
|物件儲存|S3|Blob Storage|Cloud Storage|
|關聯式資料庫|RDS / Aurora|Azure SQL / Database for PostgreSQL|Cloud SQL / AlloyDB / Spanner|
|NoSQL|DynamoDB|Cosmos DB|Firestore / Bigtable|
|資料倉儲|Redshift|Synapse Analytics|BigQuery|
|訊息佇列|SQS / SNS / Kinesis|Service Bus / Event Hubs|Pub/Sub|
|身分管理|IAM|Entra ID（前 Azure AD）+ RBAC|Cloud IAM|
|ML 平台|SageMaker / Bedrock|Azure ML / Azure OpenAI Service|Vertex AI|
|IaC|CloudFormation|ARM / Bicep|Deployment Manager（多用 Terraform）|

## 三、各家特色

**AWS**

- 服務數量最多，第三方整合與社群資源最豐富。
- Aurora、DynamoDB、Lambda 為其代表性服務。
- 缺點：服務命名與設定複雜，成本結構較難預估。

**Azure**

- 與 Active Directory、Microsoft 365、Windows Server、SQL Server 授權（Hybrid Benefit）整合深。
- 混合雲能力強（Azure Arc、Azure Stack）。
- 與 OpenAI 合作，提供 Azure OpenAI Service。

**GCP**

- Kubernetes 起源於 Google，GKE 成熟度高。
- BigQuery 為 serverless 資料倉儲，計價可依掃描量。
- 全球私有骨幹網路、Cloud Run 的 scale-to-zero 與簡潔部署體驗。
- 缺點：企業級服務與在地支援覆蓋度相對較弱。

## 四、計價與運作模式差異

- 三者皆為 pay-as-you-go，並提供 1～3 年承諾折扣：AWS Savings Plans / Reserved Instances、Azure Reservations、GCP Committed Use Discounts。
- GCP 提供 Sustained Use Discounts（自動，依使用時長折扣）。
- 三者皆有 Spot / Preemptible 類低價可中斷機器。
- 出站流量（egress）皆收費，是跨雲遷移與多雲架構的主要成本因素。

## 五、選型依據

|      | AWS                | Azure                                                          | GCP                              |
| ---- | ------------------ | -------------------------------------------------------------- | -------------------------------- |
| 核心特性 | 服務最多、選擇彈性高         | 與微軟生態深度整合                                                      | 資料分析、ML、容器化                      |
| 強項   | 服務廣度與成熟度，全球基礎設施規模大 | Entra ID（原 Azure AD）、M365、Windows Server / .NET、混合雲（Azure Arc） | BigQuery、Vertex AI、GKE、Cloud Run |
| 代價   | 服務多、組合複雜，學習與治理成本高  | 與非微軟生態整合較不順，部分服務命名與設定較繁瑣                                       | 企業級服務與市佔較小，地區與合作夥伴生態較少           |
| 典型客戶 | 新創到大型企業皆有，通用型      | 既有微軟環境的企業、政府、金融                                                | 資料密集、AI/ML、cloud-native 團隊       |
