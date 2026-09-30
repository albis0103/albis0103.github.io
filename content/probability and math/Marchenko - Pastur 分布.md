
**具體情境**

設 $X \in \mathbb{R}^{p \times n}$，$X_{ij}$ 獨立同分布，$\mathbb{E}[X_{ij}] = 0$，$\operatorname{Var}(X_{ij}) = \sigma^2$。

樣本共變異數矩陣 $S_n = \frac{1}{n}XX^\top \in \mathbb{R}^{p \times p}$，特徵值為 $\lambda_1, \dots, \lambda_p$。

真實共變異數矩陣是 $\sigma^2 I$，所有特徵值都等於 $\sigma^2$。直覺上樣本特徵值也應該接近 $\sigma^2$，但這只在 $n \gg p$ 時成立。當 $p$ 和 $n$ 同數量級時，即使資料完全是雜訊，特徵值也會明顯散開，有的遠大於 $\sigma^2$，有的遠小於 $\sigma^2$。

MP 分布就是：當 $p, n \to \infty$ 且 $p/n \to \gamma$ 時，這 $p$ 個特徵值的直方圖趨近的曲線。

**PDF**

$$  
f(x) =  
\begin{cases}  
\dfrac{\sqrt{(\lambda_+ - x)(x - \lambda_-)}}{2\pi\sigma^2\gamma x}, & \lambda_- \le x \le \lambda_+ \[2ex]  
0, & \text{otherwise}  
\end{cases}  
$$

$$  
\lambda_- = \sigma^2(1 - \sqrt{\gamma})^2, \qquad \lambda_+ = \sigma^2(1 + \sqrt{\gamma})^2  
$$

- 平均 $= \sigma^2$：$(S_n)_{ii}$ 的期望值為 $\sigma^2$，且 $\sum_i \lambda_i = \operatorname{tr}(S_n)$，所以 $\frac{1}{p}\sum_i \lambda_i$ 的期望值為 $\sigma^2$。
- 變異數 $= \sigma^4\gamma$，與 $\gamma$ 成正比。

** $\gamma$ 的影響**

- $\gamma \to 0$（$n \gg p$）：$\lambda_-, \lambda_+ \to \sigma^2$，分布縮成一點，樣本共變異數估得準。
- $\gamma$ 越大：區間越寬，估計越不準。
- $\gamma > 1$（$p > n$）：$S_n$ 秩至多為 $n$，比例 $1 - 1/\gamma$ 的特徵值恰好為 0；其餘比例 $1/\gamma$ 的特徵值按 $f(x)$ 分布（此時 $f$ 的積分為 $1/\gamma$）。

**PCA 虛構特徵向量問題**

設真實共變異數在某個方向上的變異數為 $\sigma^2(1 + \theta)$，其他方向為 $\sigma^2$，$\theta$ 表示訊號強度。

- **純雜訊（$\theta = 0$）**：沒有特殊方向，樣本特徵向量是隨機方向，換一批樣本就不同。最大特徵值被高估到 $\lambda_+$，看起來像主成分，但不可重現。
- **低於門檻（$\theta \le \sqrt{\gamma}$）**：最大特徵值停在 $\lambda_+$，與雜訊無法區分；樣本特徵向量與真實方向幾乎正交。PCA 找到的方向是虛構的，真實訊號被漏掉。
- **高於門檻（$\theta > \sqrt{\gamma}$）**：最大特徵值脫離，收斂到 $\sigma^2(1 + \theta)(1 + \gamma/\theta) > \lambda_+$，但高估了真實值 $\sigma^2(1 + \theta)$。樣本特徵向量與真實方向夾角的餘弦平方收斂到

$$  
\frac{1 - \gamma/\theta^2}{1 + \gamma/\theta}  
$$

以 $\gamma = 0.5$（門檻 $\sqrt{0.5} \approx 0.707$）為例：

|訊號強度 $\theta$|餘弦平方|夾角|
|---|---|---|
|0.7（低於門檻，極限為 0）|0|90°|
|1|1/3|≈ 55°|
|2|0.7|≈ 33°|
|10|≈ 0.95|≈ 13°|

**為什麼有用**

$\lambda_+$ 是純雜訊能產生的最大特徵值（需四階動差有限）。對真實資料做 PCA 時：

- 落在 $[\lambda_-, \lambda_+]$ 內的特徵值，和雜訊無法區分。
- 明顯大於 $\lambda_+$ 的特徵值，才代表資料中有結構。

所以 MP 分布提供了判斷「哪些主成分是訊號、哪些是雜訊」的基準。