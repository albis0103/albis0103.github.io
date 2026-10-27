問題出在 **Cloud Build 用來執行建置的服務帳戶沒有權限**，不是你的帳號。這個指令補的就是這個缺口。

**原因**

`gcloud builds submit` 其實是兩段：你的帳號把原始碼上傳，然後 Cloud Build 以一個**服務帳戶**的身分執行建置、寫日誌、把映像檔推到 Artifact Registry。Google 在 2024 年中改了新專案的預設行為：建置改用 Compute Engine 預設服務帳戶（`<專案編號>-compute@developer.gserviceaccount.com`），而且不再自動給它建置所需的角色。所以新專案這個帳戶是空的，建置一開始就被拒。

錯誤訊息只寫「The caller does not have permission」並指向你的帳號，看起來像是你的權限問題，這是誤導。我當時是依「新專案、API 與計費都正常、你應該是 owner」推測是這個原因，你貼的指令跑完就好，印證了這個判斷。

**四個角色各自的用途**

|角色|用途|
|---|---|
|`cloudbuild.builds.builder`|執行建置本身|
|`storage.objectAdmin`|讀取你上傳到 Cloud Storage 的原始碼|
|`artifactregistry.writer`|把映像檔推進 Artifact Registry|
|`logging.logWriter`|寫建置日誌|

**這和你的專案設定無關**

你的 Dockerfile、`deploy.sh` 都沒有問題，之後在同一個專案重複部署不需要再做這步。若將來換新專案，會遇到同樣的錯誤，補同樣的四個角色即可。

這些角色是授給預設服務帳戶的專案層級權限，範圍比嚴格最小權限大。對這個課程展示可以接受，展示結束後若要收斂，可以再把角色移除。