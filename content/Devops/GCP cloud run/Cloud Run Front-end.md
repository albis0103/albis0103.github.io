![[Pasted image 20261012232316.png]]


### Ingress Check
: check IP 是否合法
#### Setting
- `All traffic`：全部放行
- `Internal`：僅限GCP VPC內 VM
- `Internal and Cloud Load Balenciung`: 內部 ＋ 通過 Cloud Balencer 檢查的


#### IAM Invoker 權限（`roles/run.invoker`）
：檢查HTTP request是否具備權限
- Case: Public Web APP：給Browser（一般user）存取
	- Role: `roles/run.invoker`
	- Member: `allUsers`
- Case: Load Balencer IAM
	- Reverse Proxy / Load Balencer 轉發請求時戴上 OIDC ID Token / Service Account驗證標頭

#### Revision與Tag Traffic Split
：依設定好版分比分流不同 Revisions
- Traffic Split
	- ex. $90\%\;\rightarrow$ Stable Revision (ex. `v1-0-0`), $10\%\;\rightarrow$ Caanary Revision (ex. `v1-1-0`)