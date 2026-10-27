

## 定義

Hamming space 是由長度為 n 的 q 元字串所構成的集合，配上 Hamming distance 作為度量：

$$\mathcal{H}(n, q) = {0, 1, \dots, q-1}^n, \quad d_H(x, y) = \left|{ i : x_i \neq y_i }\right|$$

最常見的是二元情形 q = 2，即 {0,1}^n（n 維二元超立方體的頂點集合），因此 |H(n,2)| = 2^n。

## 度量性質

d_H 滿足度量公理：

- 非負性與同一性：d_H(x, y) ≥ 0，且 d_H(x, y) = 0 ⇔ x = y
- 對稱性：d_H(x, y) = d_H(y, x)
- 三角不等式：d_H(x, z) ≤ d_H(x, y) + d_H(y, z)

二元情形下 d_H(x, y) = wt(x ⊕ y)，即 XOR 後的 Hamming weight（popcount），硬體上可用 XOR + POPCNT 指令在單一 word 內計算。

## Hamming ball

以 x 為中心、半徑 r 的球：

$$B_r(x) = { y \in \mathcal{H}(n,q) : d_H(x,y) \le r }$$

其大小與中心無關：

$$|B_r| = \sum_{i=0}^{r} \binom{n}{i} (q-1)^i$$

## 在編碼理論的角色

碼 C ⊆ H(n,q) 的最小距離 d = min_{x≠y∈C} d_H(x,y)：

- 可偵測 d−1 個錯誤
- 可糾正 t = ⌊(d−1)/2⌋ 個錯誤（半徑 t 的球彼此不相交）

**Hamming bound（sphere-packing bound）**：

$$|C| \cdot \sum_{i=0}^{t}\binom{n}{i}(q-1)^i \le q^n$$

等號成立的碼稱為 perfect code，例如 Hamming code（n = 2^m − 1, k = n − m, d = 3）與 binary Golay code。

## 在檢索與機器學習的角色

- **Binary hashing / Learning to Hash**：把高維向量映射成 n-bit 二元碼（如 LSH、ITQ、Spectral Hashing），使原空間相似度對應到 Hamming distance 小，近似最近鄰搜尋轉為 Hamming space 上的搜尋。
- **Multi-Index Hashing (MIH)**：將 n-bit 碼切成 m 段子字串，依鴿籠原理，若 d_H(x,y) ≤ r，則至少有一段子字串的距離 ≤ ⌊r/m⌋，據此建 m 張 hash table 做精確 r-neighbor 搜尋。
- **Binary quantization / 向量資料庫**：embedding 以符號函數二值化（sign(x_i)），距離計算降為 XOR + popcount，記憶體為 float32 的 1/32；常搭配 rescoring 補回精度。
- 其他：DNA storage 與 barcode 設計、locality-sensitive hashing 中 bit sampling 對應 Hamming distance 的 LSH 族。

## 重要性質與限制

- **維度災難**：n 大時，隨機點對的距離集中於 n/2 附近（二項分布，標準差 √n/2），距離區分度下降。
- **只計算位置不同的數量**：不考慮 insertion/deletion（那是 Levenshtein distance），也不考慮符號之間的語意距離。
- **二元情形的等價關係**：d_H(x,y) 等於 {0,1}^n 超立方體圖上的最短路徑長度（圖度量，L1 距離在 {0,1}^n 上亦等於 d_H，而 L2² 同樣等於 d_H）。

需要的話，我可以進一步推導 Hamming bound、MIH 的搜尋複雜度，或 Hamming code 的 parity-check 矩陣構造。