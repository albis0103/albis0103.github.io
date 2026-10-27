
![[Pasted image 20261007094100.png]]

Bloom filter 是一種**空間效率極高的機率型資料結構**，用於membership query：元素 x 是否 $\in$ 集合 S。$$\text{it only return 'may be' or 'no'}$$


- 回答「不在集合中」時**一定正確**（無 false negative）
- 回答「在集合中」時**可能錯誤**（有 false positive）
- 標準版本不支援刪除


### 資料結構

：長度為 $m$ 的 Bit Array（初始全為 0）， $k$ 個獨立的Hash Function $h_1, …, h_k$，每個映射到 $\{0, …, m−1\}$。

- **Insert(x)**：將位置 $h_1(x), …, h_k(x)$ 的位元設為 1
- **Query(x)**：檢查$h_(x), …, h_k(x)$ 是否**全為 1**
    - 任一位元為 0 → x 必定不在 S 中
    - 全為 1 → x 可能在 S 中

False positive 來源：其他元素插入時將這 k 個位置恰好都設為 1（hash collision 的累積）。

## 複雜度

|操作|時間|空間|
|---|---|---|
|Insert|O(k)|O(m)|
|Query|O(k)|—|

與元素本身大小無關，空間僅取決於 n 與 p。

## 誤判率分析

設已插入 n 個元素，假設雜湊函數均勻且獨立。

某一特定位元在 n 次插入後仍為 0 的機率：(特定bit被選中$=1$機率: $\frac{1}{m}$)
by [[tayler expansion]] $e^x=\lim_{n \rightarrow \infty}(1+\frac{x}{n})^n$ 

$$\left(1 - \frac{1}{m}\right)^{kn} \approx [e^{-1}]^{\frac{kn}{m}}=e^{-\frac{kn}{m}}$$

false positive 機率 p（k 個位置皆為 1）：

$$p \approx \left(1 - e^{-\frac{kn}{m}}\right)^k$$

**最佳雜湊函數數量**（對 k 微分求極小值）：

$$k^* = \frac{m}{n}\ln 2$$

此時位元陣列恰有一半為 1，且：

$$p = 2^{-k^*} = (0.6185)^{m/n}$$

**給定目標誤判率 p 所需的位元數**：

$$m = -\frac{n \ln p}{(\ln 2)^2} \approx 1.44, n \log_2(1/p)$$

例：p = 1% 時，每個元素約需 9.6 bits，k ≈ 7。



## 變體

- **Counting Bloom filter**：每個位置改用計數器（通常 4 bits），支援刪除，空間約 4 倍
- **Scalable Bloom filter**：n 未知時，以多層遞增大小的 filter 動態擴充，並控制總誤判率
- **Cuckoo filter**：支援刪除，在低誤判率（< 3%）下空間效率常優於 Bloom filter
- **Blocked Bloom filter**：把 k 個位元限制在單一 cache line 內，改善 CPU cache 局部性

## 典型應用

- **LSM-tree 資料庫**（RocksDB、Cassandra、HBase、LevelDB）：查詢前以 Bloom filter 判斷 key 是否可能存在於某個 SSTable，避免不必要的磁碟讀取
- **快取穿透防護**：在 cache/DB 前置 filter，攔截一定不存在的 key
- **網路安全**：惡意 URL / 已知 malware hash 的快速預篩（Chrome Safe Browsing 早期設計即採用類似機制）
- **爬蟲去重**：判斷 URL 是否已爬取
- **分散式系統**：Bigtable、Squid proxy cache digest 等

## 使用限制

1. 誤判率隨 n 超過設計容量而急遽上升，需事先估計 n
2. 無法列舉或取出元素
3. 標準版本不可刪除（把位元清 0 會影響其他元素，產生 false negative）
4. 適用情境須能容忍 false positive，或有後續精確驗證步驟（如讀取磁碟確認）

若需要，我可以補充程式實作（Python）或與 Cuckoo filter 的詳細比較。