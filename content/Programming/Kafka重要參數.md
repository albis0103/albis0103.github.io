以下以 Kafka 3.x（含 KRaft 模式）為準，依元件分類。

## Broker 端

|參數|說明|
|---|---|
|`broker.id` / `node.id`|節點唯一 ID。KRaft 模式使用 `node.id`。|
|`process.roles`|KRaft 專用，`broker`、`controller` 或兩者並存。|
|`listeners` / `advertised.listeners`|前者為 broker 實際綁定的位址；後者為回報給 client 的連線位址，容器或 NAT 環境下設定錯誤是最常見的連線失敗原因。|
|`log.dirs`|分區資料目錄，可設多個磁碟路徑。|
|`num.partitions`|自動建立 topic 時的預設分區數。|
|`default.replication.factor`|自動建立 topic 時的預設副本數。|
|`auto.create.topics.enable`|生產環境建議設為 `false`。|
|`num.network.threads` / `num.io.threads`|處理網路請求與磁碟 I/O 的執行緒數。|

## Topic / 副本與持久性

|參數|說明|
|---|---|
|`min.insync.replicas`|ISR 中至少需有幾個副本，`acks=all` 的寫入才會成功。與 `replication.factor=3` 搭配常設為 2。|
|`unclean.leader.election.enable`|允許非 ISR 副本成為 leader。設為 `true` 可提高可用性，但可能遺失資料，預設 `false`。|
|`replica.lag.time.max.ms`|follower 超過此時間未追上 leader 即被移出 ISR。|
|`retention.ms` / `retention.bytes`|依時間或大小保留資料，預設 7 天。|
|`segment.bytes` / `segment.ms`|log segment 滾動條件。保留與刪除以 segment 為單位。|
|`cleanup.policy`|`delete` 或 `compact`（依 key 保留最新值），可同時指定。|
|`max.message.bytes`|單一 batch 大小上限，需與 producer 的 `max.request.size`、consumer 的 `fetch.max.bytes` 配合調整。|

## Producer

|參數|說明|
|---|---|
|`acks`|`0` 不等待確認；`1` 等 leader 寫入；`all`（`-1`）等所有 ISR 確認。3.0 起預設為 `all`。|
|`enable.idempotence`|啟用冪等性，避免重試造成重複，3.0 起預設 `true`。|
|`retries` / `delivery.timeout.ms`|現行版本以 `delivery.timeout.ms`（預設 120 秒）為發送總時限，`retries` 預設為極大值。|
|`max.in.flight.requests.per.connection`|單連線未確認請求數。啟用冪等性時 ≤ 5 仍可保證順序。|
|`batch.size`|每個分區的 batch 大小上限（byte）。|
|`linger.ms`|等待湊 batch 的時間，調大可提升吞吐、增加延遲。|
|`compression.type`|`none`、`gzip`、`snappy`、`lz4`、`zstd`。|
|`buffer.memory`|producer 端緩衝總記憶體，滿了會阻塞至 `max.block.ms`。|
|`transactional.id`|啟用交易，搭配 exactly-once 語意。|

## Consumer

|參數|說明|
|---|---|
|`group.id`|consumer group 識別。|
|`enable.auto.commit` / `auto.commit.interval.ms`|自動提交 offset。需精確控制處理語意時應關閉並手動提交。|
|`auto.offset.reset`|無已提交 offset 時的行為：`earliest`、`latest`、`none`。|
|`max.poll.records`|單次 `poll()` 回傳的最大筆數。|
|`max.poll.interval.ms`|兩次 `poll()` 的最大間隔，超過則被視為失效並觸發 rebalance。處理時間長時須調大或減少 `max.poll.records`。|
|`session.timeout.ms` / `heartbeat.interval.ms`|心跳逾時與頻率，後者通常設為前者的 1/3 以內。|
|`fetch.min.bytes` / `fetch.max.wait.ms`|broker 累積足夠資料或等待逾時後才回應 fetch。|
|`max.partition.fetch.bytes`|每個分區單次 fetch 的上限。|
|`partition.assignment.strategy`|分區分配策略，如 `RangeAssignor`、`CooperativeStickyAssignor`。後者支援增量 rebalance。|
|`isolation.level`|`read_committed` 僅讀取已提交的交易訊息。|
|`group.instance.id`|靜態成員資格，可減少重啟造成的 rebalance。|

## 常見調校方向

- **不遺失資料**：`acks=all`、`replication.factor=3`、`min.insync.replicas=2`、`unclean.leader.election.enable=false`、`enable.idempotence=true`。
- **高吞吐**：增大 `batch.size` 與 `linger.ms`、啟用 `lz4` 或 `zstd` 壓縮、增加分區數。
- **低延遲**：`linger.ms=0`、避免過大的 `fetch.min.bytes`。
- **減少 rebalance**：合理設定 `max.poll.interval.ms`、使用 `CooperativeStickyAssignor` 與靜態成員資格。

需要的話，我可以針對特定情境（例如吞吐量優化或 exactly-once 配置）給出完整的參數組合。




這是單節點 KRaft 模式（無 ZooKeeper）的 Kafka，broker 與 controller 合併在同一個容器。Confluent 映像檔會將 `KAFKA_` 前綴的環境變數轉換為 `server.properties`，例如 `KAFKA_NODE_ID` → `node.id`，底線轉為點。

## 基本設定

|項目|說明|
|---|---|
|`image: cp-kafka:7.5.0`|Confluent Platform 7.5.0，對應 Apache Kafka 3.5.x。|
|`ports: "9092:9092"`|只暴露 `PLAINTEXT_HOST` 這個 listener 給 host。|
|`restart: "no"`|容器異常退出不自動重啟。|

## KRaft 核心

|變數|說明|
|---|---|
|`CLUSTER_ID` / `KAFKA_CLUSTER_ID`|叢集 ID（base64 UUID，22 字元）。同一份設定重複宣告，實際上只需要 `CLUSTER_ID` 即可，容器啟動腳本用它執行 `kafka-storage format`。|
|`KAFKA_NODE_ID: 1`|節點 ID。|
|`KAFKA_PROCESS_ROLES: 'broker,controller'`|此節點同時擔任兩種角色，即 combined mode。|
|`KAFKA_CONTROLLER_QUORUM_VOTERS: '1@kafka:29093'`|controller quorum 成員，格式 `nodeId@host:port`。此處 `kafka` 為服務名稱，由 Docker DNS 解析。|
|`KAFKA_CONTROLLER_LISTENER_NAMES: 'CONTROLLER'`|指定哪個 listener 用於 controller 通訊。|

`node.id` 必須與 quorum voters 中的 ID 一致（此處皆為 1）。

## Listener 配置

三個 listener，各有分工：

|Listener 名稱|綁定|對外宣告位址|用途|
|---|---|---|---|
|`PLAINTEXT`|`:29092`|`kafka:29092`|Docker 網路內部的 client（其他容器）及 inter-broker 通訊|
|`CONTROLLER`|`:29093`|不宣告|controller quorum 通訊|
|`PLAINTEXT_HOST`|`:9092`|`localhost:9092`|host 機器上的 client|

- **`KAFKA_LISTENERS`**：broker 實際綁定的位址。`:` 前省略 host，代表綁定所有網卡。
- **`KAFKA_ADVERTISED_LISTENERS`**：metadata 回傳給 client 的位址，client 之後會用這個位址連線。`CONTROLLER` listener 不需要列在這裡。
- **`KAFKA_LISTENER_SECURITY_PROTOCOL_MAP`**：listener 名稱對應的安全協定。名稱是自訂的，所以必須明確對應。這裡全部都是 `PLAINTEXT`（無加密、無認證）。
- **`KAFKA_INTER_BROKER_LISTENER_NAME`**：broker 之間使用 `PLAINTEXT` listener。單節點下實際不會用到，但仍為必要設定。

### 連線方式

- 同一個 compose 網路內的服務：`kafka:29092`
- host 上的程式（本機開發、IDE）：`localhost:9092`

兩者必須分開，因為 client 連上後會依 advertised listener 重新連線。若只宣告 `localhost:9092`，容器內的 client 會連到自己的 localhost 而失敗。

## 穩定性配置

單節點下必須把內部 topic 的副本數調為 1：

|變數|預設|說明|
|---|---|---|
|`KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1`|3|`__consumer_offsets` 的副本數。|
|`KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1`|3|`__transaction_state` 的副本數。|
|`KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1`|2|交易 log 的最小 ISR。|
|`KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0`|3000|group 首次 rebalance 的等待時間，設為 0 可加快啟動。|

若未調整，只有一個 broker 時，consumer group 與 transaction 功能會因為無法滿足副本數而失敗。

## 注意事項

1. **無資料持久化**：沒有設定 `volumes`，容器刪除後 topic 與資料全部消失。需保留資料時，應掛載 `/var/lib/kafka/data`。
2. **`restart: "no"`**：配合無 volume 的設定，適合開發或測試，不適合長期運行。
3. **`CLUSTER_ID` 重複宣告**：`CLUSTER_ID` 與 `KAFKA_CLUSTER_ID` 重複，可擇一。
4. **`container_name: kafka-broker` 與 quorum voters 的 `kafka`**：voters 使用的是 compose 服務名稱 `kafka`（假設 service key 為 `kafka`），而非 `container_name`。若 service key 改名，voters 位址必須同步修改。
5. **未設定的行為**：`auto.create.topics.enable` 預設為 `true`，producer 寫入不存在的 topic 時會自動建立，且分區數預設為 1。
6. **僅限開發環境**：單節點、無副本、無認證、無加密，不具備容錯能力。正式環境至少需 3 個 controller 與 3 個 broker，並啟用 SASL/TLS。