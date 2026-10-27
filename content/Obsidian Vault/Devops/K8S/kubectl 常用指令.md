
## 叢集與環境

| 指令                                                      | 用途                                                  |
| ------------------------------------------------------- | --------------------------------------------------- |
| `kubectl cluster-info`                                  | 顯示 API Server、CoreDNS 等控制平面端點，確認連得到叢集               |
| `kubectl version`                                       | 顯示 client 與 server 的版本                              |
| `kubectl config get-contexts`                           | 列出 kubeconfig 中所有 context（叢集 + 使用者 + namespace 的組合） |
| `kubectl config use-context <context>`                  | 切換目前操作的叢集                                           |
| `kubectl config set-context --current --namespace=<ns>` | 設定目前 context 的預設 namespace，之後不必每次加 `-n`             |
| `kubectl get nodes -o wide`                             | 列出節點，並顯示 IP、OS、kernel、container runtime             |

## 查看資源

|指令|用途|
|---|---|
|`kubectl get pods`|列出 Pod。`-n` 指定 namespace，`-A` 所有 namespace，`-o wide` 顯示節點與 IP，`-w` 持續監看變化|
|`kubectl get deploy,svc,ing,cm,secret`|一次列出多種資源（Deployment、Service、Ingress、ConfigMap、Secret）|
|`kubectl get all`|列出 namespace 內主要工作負載資源（不含 ConfigMap、Secret、Ingress 等）|
|`kubectl get pod <pod> -o yaml`|輸出該 Pod 完整的 YAML（含 status）|
|`kubectl describe pod <pod>`|顯示詳細資訊與 Events，排查 Pending、CrashLoopBackOff、ImagePullBackOff 時最常用|
|`kubectl api-resources`|列出叢集支援的所有資源類型、縮寫與是否屬於 namespace|
|`kubectl explain pod.spec.containers`|查詢欄位的說明文件，寫 YAML 時用|

## 建立 / 更新 / 刪除

|指令|用途|
|---|---|
|`kubectl apply -f manifest.yaml`|宣告式建立或更新資源，使其符合 YAML 內容|
|`kubectl apply -k <dir>`|套用 Kustomize 目錄|
|`kubectl create deployment web --image=nginx --replicas=2`|命令式建立 Deployment|
|`kubectl delete -f manifest.yaml`|刪除 YAML 中定義的所有資源|
|`kubectl delete pod <pod>`|刪除 Pod；若由 Deployment 管理，會被重建|
|`kubectl edit deploy <name>`|以編輯器直接修改線上資源的設定|
|`kubectl patch deploy <name> -p '...'`|以 JSON 片段局部修改欄位（此例將 replicas 改為 3）|
|`kubectl diff -f manifest.yaml`|套用前預覽本地 YAML 與線上狀態的差異|

## Deployment 管理

|指令|用途|
|---|---|
|`kubectl scale deploy <name> --replicas=5`|手動調整副本數|
|`kubectl set image deploy/<name> <container>=<image>:<tag>`|更新容器 image，觸發滾動更新|
|`kubectl rollout status deploy/<name>`|觀察滾動更新是否完成|
|`kubectl rollout history deploy/<name>`|查看版本歷史|
|`kubectl rollout undo deploy/<name>`|回滾到上一版（`--to-revision=N` 指定版本）|
|`kubectl rollout restart deploy/<name>`|重啟所有 Pod（逐步替換），常用於重新載入 ConfigMap/Secret|
|`kubectl autoscale deploy <name> --min=2 --max=10 --cpu-percent=70`|建立 HPA，CPU 使用率超過 70% 時在 2～10 個副本間自動擴縮|

## 除錯

|指令|用途|
|---|---|
|`kubectl logs <pod>`|查看容器日誌。`-c` 指定容器，`-f` 持續追蹤，`--previous` 看上一次崩潰的容器，`--tail=100` 只看最後 100 行|
|`kubectl exec -it <pod> -- /bin/sh`|進入容器開啟互動式 shell|
|`kubectl port-forward pod/<pod> 8080:80`|將本機 8080 轉發到 Pod 的 80 port，用於本地測試|
|`kubectl port-forward svc/<svc> 8080:80`|同上，但轉發到 Service|
|`kubectl cp <pod>:/path/file ./file`|在本機與 Pod 之間複製檔案|
|`kubectl get events --sort-by=.metadata.creationTimestamp`|依時間排序列出叢集事件，查看排程失敗、探針失敗等|
|`kubectl top pod` / `kubectl top node`|查看 CPU、記憶體使用量（需安裝 metrics-server）|
|`kubectl run tmp --rm -it --image=busybox -- sh`|啟動臨時 Pod 測試網路（如 `wget`、`nslookup`），離開後自動刪除|
|`kubectl debug <pod> -it --image=busybox --target=<container>`|對既有 Pod 附加臨時除錯容器，適用於沒有 shell 的精簡 image|

## Namespace / 節點

|指令|用途|
|---|---|
|`kubectl get ns`|列出 namespace|
|`kubectl create ns <ns>`|建立 namespace|
|`kubectl cordon <node>`|將節點標記為不可排程，現有 Pod 不受影響|
|`kubectl drain <node> --ignore-daemonsets --delete-emptydir-data`|驅逐節點上的 Pod 以便維護；`--ignore-daemonsets` 略過 DaemonSet Pod，`--delete-emptydir-data` 允許刪除 emptyDir 資料|
|`kubectl uncordon <node>`|恢復節點可排程|
|`kubectl taint nodes <node> key=value:NoSchedule`|為節點加上 taint，沒有對應 toleration 的 Pod 不會被排程上去|
|`kubectl label node <node> key=value`|為節點加標籤，配合 nodeSelector / affinity 使用|

## 設定 / 權限

|指令|用途|
|---|---|
|`kubectl create configmap <name> --from-file=<file>`|由檔案建立 ConfigMap|
|`kubectl create secret generic <name> --from-literal=key=value`|以鍵值建立 Secret（內容僅 base64 編碼，非加密）|
|`kubectl auth can-i create pods --as=<user>`|檢查某使用者是否有建立 Pod 的 RBAC 權限|
|`kubectl get sa,role,rolebinding,clusterrole,clusterrolebinding`|列出 ServiceAccount 與 RBAC 相關資源|

## 常用技巧

|指令|用途|
|---|---|
|`-o jsonpath='{.items[*].metadata.name}'`|以 JSONPath 擷取特定欄位，適合腳本|
|`-l app=web`|以 label selector 篩選資源|
|`--field-selector=status.phase=Running`|以欄位值篩選（如 Pod 狀態）|
|`-o yaml > pod.yaml`|匯出資源 YAML 存檔|
|`--dry-run=client -f manifest.yaml`|僅本地驗證，不送到叢集|
|`create deploy ... --dry-run=client -o yaml`|產生 YAML 範本，是撰寫 manifest 的快速方法|
|`alias k=kubectl`|縮短指令輸入|

## 補充：`apply` 與 `create` 的差異

- `create` 是命令式，資源已存在就報錯。
- `apply` 是宣告式，不存在就建立，已存在就更新，適合版本控制與重複部署。