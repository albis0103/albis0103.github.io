
- input : ${f(\mathbf{x}) : \mathbf{x} \in \mathcal{X}} \iff \mathbf{x}_1, \dots, \mathbf{x}_n \in \mathcal{X}$
- 高斯過程 ${f(\mathbf{x}) : \mathbf{x} \in \mathcal{X}} \iff \mathbf{x}_1, \dots$
- 隨機向量 $(f(\mathbf{x}_1), \dots, f(\mathbf{x}_n))^\top$ 服從多元高斯分布。

GP 完全由兩個函數決定：

- 均值(mean)函數：$m(\mathbf{x}) = \mathbb{E}[f(\mathbf{x})]$
- 共變異(Covariance)函數（Kernel Function）：$k(\mathbf{x}, \mathbf{x}') = \mathbb{E}\big[(f(\mathbf{x}) - m(\mathbf{x}))(f(\mathbf{x}') - m(\mathbf{x}'))\big]$

記為 $f \sim \mathcal{GP}(m, k)$。

多元高斯分布是有限維隨機向量的分布，GP 是它在**函數空間**（無限維）上的推廣。GP 就是函數的機率分布。一致性由 Kolmogorov 擴張定理保證：高斯分布的邊際仍是高斯，所以對任意子集取邊際，結果都相容。

核函數 $k$ 必須是**正半定**（positive semi-definite）。也就是說，對任意 ${\mathbf{x}_i}_{i=1}^n$，Gram 矩陣 $K_{ij} = k(\mathbf{x}_i, \mathbf{x}_j)$ 滿足 $K \succeq 0$。

## 2. 核函數決定函數性質

核函數編碼了對 $f$ 的先驗假設，包括平滑度、週期性和尺度。常見核函數如下：

| 核函數                       | 形式                                                                                    | 樣本函數性質                                                           |
| ------------------------- | ------------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| Squared Exponential (RBF) | $\sigma_f^2 \exp!\left(-\dfrac{\lVert\mathbf{x}-\mathbf{x}'\rVert^2}{2\ell^2}\right)$ | 無限次可微                                                            |
| Matérn-$\nu$              | 含 Bessel 函數 $K_\nu$                                                                   | $\lceil\nu\rceil - 1$ 次均方可微；$\nu = 1/2$ 對應 Ornstein–Uhlenbeck 過程 |
| Periodic                  | $\sigma_f^2 \exp\left(-\dfrac{2\sin^2(\pi\lvert x-x'\rvert/p)}{\ell^2}\right)$        | 週期為 $p$                                                          |
| Linear                    | $\sigma_b^2 + \sigma_v^2, \mathbf{x}^\top\mathbf{x}'$                                 | 等價於貝氏線性迴歸                                                        |

參數說明：

- $\ell$（length-scale）控制相關性隨距離衰減的速度。$\ell$ 越大，函數越平滑。
- $\sigma_f^2$ 控制輸出的振幅。
- 核函數的和與乘積仍是合法核函數，可以組合出複合結構（例如 趨勢 + 週期）。

## 3. GP 迴歸：後驗推導
https://roger010620.medium.com/高斯過程-gaussion-process-含python實作-642965af8b2
**觀測模型**：$y_i = f(\mathbf{x}_i) + \varepsilon_i$，其中 $\varepsilon_i \sim \mathcal{N}(0, \sigma_n^2)$ 為 i.i.d. 雜訊，先驗為 $f \sim \mathcal{GP}(0, k)$。

**記號**：

- 訓練輸入 $X \in \mathbb{R}^{n \times d}$，觀測值 $\mathbf{y} \in \mathbb{R}^n$，測試輸入$X_*$（給一組新樣本預測）
- $K = k(X, X)$
- $K_* = k(X, X_*)$
- $K_{**} = k(X_*, X_*)$

**聯合分布**（依 GP 定義）：觀測值跟新預測值的Joint Distribution
$$  
\begin{bmatrix} \mathbf{y} \\ \mathbf{f}_* \end{bmatrix}  
\sim \mathcal{N}\left( \mathbf{0}, 
\begin{bmatrix}\operatorname{Cov}(\mathbf{y},\mathbf{y}) & \operatorname{Cov}(\mathbf{y},\mathbf{f}_*)\\ \operatorname{Cov}(\mathbf{f}_*,\mathbf{y}) & \operatorname{Cov}(\mathbf{f}_*,\mathbf{f}_*)\end{bmatrix} \right)  =\mathcal{N}\left( \mathbf{0}, 
\begin{bmatrix} K + \sigma_n^2 I & K_* \\ K_*^\top & K_{**} \end{bmatrix} \right)  
$$
$$\operatorname{Cov}(\mathbf{y},\mathbf{y}) = \operatorname{Cov}(\mathbf{f}+\varepsilon,\ \mathbf{f}+\varepsilon) = \operatorname{Cov}(\mathbf{f},\mathbf{f}) + \operatorname{Cov}(\mathbf{f},\varepsilon) + \operatorname{Cov}(\varepsilon,\mathbf{f}) + \operatorname{Cov}(\varepsilon,\varepsilon)$$
- 因 ε ⟂ f：中間兩項為 0
- $\operatorname{Cov}(\mathbf{f},\mathbf{f}) = K$（由 k 定義），$\operatorname{Cov}(\varepsilon,\varepsilon)=\sigma_n^2 I$
- 所以 $= K + \sigma_n^2 I$

$p(f(X^*)|Y,X,X^*) \sim N(\mu^*,\Sigma^*)$ , 求$\mu^*, \Sigma^*$


**條件分布**：套用多元高斯的條件分布公式。
[[Gaussian Distribution]]
- Conditional Mean: $\boldsymbol{\mu}_{A\vert{}B} = \boldsymbol{\mu}_A + \boldsymbol{\Sigma}_{AB} \boldsymbol{\Sigma}_{BB}^{-1} (\mathbf{x}_B - \boldsymbol{\mu}_B)$
- Conditional Covariance: $\boldsymbol{\Sigma}_{A\vert{}B} = \boldsymbol{\Sigma}_{AA} - \boldsymbol{\Sigma}_{AB} \boldsymbol{\Sigma}_{BB}^{-1} \boldsymbol{\Sigma}_{BA}$

$$X=(X_a,X_b)^T,\mu=(\mu_a,\mu_b^T),\Sigma=\begin{pmatrix}\Sigma_{aa} & \Sigma_{ab}\\\Sigma_{ba}&\Sigma_{bb}\end{pmatrix}$$
$$\mu_{b|a} = \Sigma_{ba}\Sigma_{aa}^{-1}(X_a-\mu_a)+\mu_b$$
$$\Sigma_{b|a}=\Sigma_{bb}-\Sigma_{ba}\Sigma_{aa}^{-1}\Sigma_{ab}$$
so
$$\mu^*=k(X^*,X)(k(X,X+\sigma^2I)^{-1}\mathbb{y}$$
$$\Sigma^*=k(X^*,X^*)-(X^*,X)(k(X,X+\sigma^2I)^{-1}k(X,X^*)$$




**解讀**：

- **後驗均值**是核函數的線性組合：令 $\boldsymbol{\alpha} = (K + \sigma_n^2 I)^{-1}\mathbf{y}$，則 $\bar{f}(\mathbf{x}_*) = \sum_{i=1}^n \alpha_i, k(\mathbf{x}_i, \mathbf{x}_*)$。這與 Representer Theorem 的形式一致，數值上等同於 kernel ridge regression 的解。
- **後驗變異數**等於先驗變異數 $K_{**}$ 減去觀測帶來的資訊量。它**不依賴 $\mathbf{y}$**，只取決於輸入點的位置。測試點離訓練資料越遠，$K_*$ 越接近 0，變異數就越接近先驗值 $\sigma_f^2$。這就是 GP 提供校準後不確定性的來源。
- 預測 $y_*$（含雜訊）時，共變異數再加上 $\sigma_n^2 I$。

## 4. 超參數學習：邊際概似

超參數 $\theta = (\ell, \sigma_f, \sigma_n, \dots)$ 透過最大化對數邊際概似求得。$f$ 已被積分掉：

$$  
\log p(\mathbf{y} \mid X, \theta) = -\tfrac{1}{2}\mathbf{y}^\top K_y^{-1}\mathbf{y} - \tfrac{1}{2}\log\lvert K_y\rvert - \tfrac{n}{2}\log 2\pi,  
\qquad K_y = K + \sigma_n^2 I  
$$

- 第一項是**資料擬合**。
- 第二項是**複雜度懲罰**：模型越彈性，$\lvert K_y\rvert$ 越大。

兩項的權衡構成自動的 Occam's razor。

梯度為：

$$  
\frac{\partial}{\partial \theta_j}\log p(\mathbf{y}\mid X,\theta) = \tfrac{1}{2}\operatorname{tr}!\left[(\boldsymbol{\alpha}\boldsymbol{\alpha}^\top - K_y^{-1})\frac{\partial K_y}{\partial \theta_j}\right]  
$$

實務上用 L-BFGS 等梯度方法最佳化。目標函數非凸，通常需要多次隨機初始化。

## 5. 與貝氏線性迴歸的關係（weight-space view）

設 $f(\mathbf{x}) = \phi(\mathbf{x})^\top \mathbf{w}$，先驗 $\mathbf{w} \sim \mathcal{N}(\mathbf{0}, \Sigma_p)$。則：

$$  
\mathbb{E}[f(\mathbf{x})] = 0, \qquad \operatorname{cov}(f(\mathbf{x}), f(\mathbf{x}')) = \phi(\mathbf{x})^\top \Sigma_p, \phi(\mathbf{x}')  
$$

所以這就是一個 GP，核函數為 $k(\mathbf{x}, \mathbf{x}') = \phi(\mathbf{x})^\top\Sigma_p\phi(\mathbf{x}')$。

反過來，給定一個核函數，根據 Mercer 定理，它對應一個（可能無限維的）特徵映射 $\phi$。例如 RBF 核對應無限多個基底函數。這就是 kernel trick 在機率框架下的版本：GP 讓我們在無限維特徵空間做貝氏推論，而計算量只與樣本數 $n$ 有關。

相關結果：單隱藏層、寬度 $\to \infty$、權重 i.i.d. 的神經網路，其先驗收斂到 GP（Neal, 1996）。這延伸出 NNGP 與 Neural Tangent Kernel 的理論。

## 6. 計算複雜度與限制

- 訓練需要分解 $K_y$，通常用 Cholesky 分解 $K_y = LL^\top$ 以求數值穩定：時間 $O(n^3)$，記憶體 $O(n^2)$。
- 預測時，均值 $O(n)$、變異數 $O(n^2)$（每個測試點）。
- 當 $n \gtrsim 10^4$ 時需要近似方法：
    - **Inducing points / Sparse GP**：SoR、FITC、SVGP（Titsias 2009 的變分法；Hensman 2013 支援 mini-batch）
    - **隨機特徵**：Random Fourier Features
    - **結構化方法**：KISS-GP、SKI，利用 Toeplitz 或 Kronecker 結構
- **非高斯概似**（例如分類）時後驗沒有解析解，需要 Laplace 近似、EP 或變分推論。
- 在高維輸入下，平穩核（只依賴 $\mathbf{x} - \mathbf{x}'$）的距離度量會失效，需要 ARD（每個維度各自一個 $\ell_d$）或結合深度特徵（Deep Kernel Learning）。

## 7. 主要應用

- **貝氏最佳化**：用 GP 作為目標函數的代理模型，再以 acquisition function（EI、UCB）利用後驗均值與變異數，在探索與利用之間取捨。
    - 你報告的 DOT（DB knob tuning）屬於這個問題類別。OtterTune 等資料庫調參系統就用 GP-based BO 作為 baseline，可以拿來做對比。
- **空間統計**：kriging 本質上就是 GP 迴歸。
- **時間序列插值、代理模型（surrogate modeling）、小樣本且需要不確定性量化的迴歸問題。**

標準參考書：Rasmussen & Williams, _Gaussian Processes for Machine Learning_ (2006)，第 2、5 章分別對應上面的第 3、4 節。