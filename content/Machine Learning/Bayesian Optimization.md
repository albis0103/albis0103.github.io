



Bayesian Optimization（BO）$x \in\mathbb{R}^d$, 通常$d \lesssim 20$，處理最佳化問題：

$$  
x^* = \arg\max_{x \in \mathcal{X}} f(x), \quad \mathcal{X} \subset \mathbb{R}^d  
$$

其中 $f$ 具有以下性質：

- **$f$ is Black-Box**：沒有解析形式，也沒有梯度資訊。
- **評估昂貴**：每次評估 $f(x)$ 的成本高，例如訓練一次模型，或跑一次資料庫 benchmark。
- **可能有雜訊**：觀測值為 $y = f(x) + \varepsilon$，其中 $\varepsilon \sim \mathcal{N}(0, \sigma_\varepsilon^2)$。


BO 的目標是在評估次數 $n$ 很小（數十到數百次）的限制下找到接近 $x^*$ 的解。
![[Pasted image 20260930231103.png]]

BO 由兩個部分組成：

1. **Surrogate model**：對 $f$ 建立機率模型，預測$\mu_n(x), \sigma^2_n(x)$
	- Time Complexity : $O(n^3)$ / Space Complexity : $O(n^2)$
		- $n^3$ from $\Sigma_0(x_{1:n},x_{1:n})$
2. **Acquisition function**：根據該模型決定下一個評估點。
	- Exploitation: 選擇目前表現最好附近（high $\mu_n$ ）
	- Exploration: 選擇不確定性高區域（High $\sigma^2_n$）
	- Time Complexity : $O(M\cdot d \cdot n^2)$
		- $M = |\mathcal{D}_n|,\;x \in \mathbb{R}^d, x_1,\;\cdots ,x_n$ 

完整 BO 迴圈

1. **用 $\mathcal{D}_n$ (Sample of $\mathcal{X}$)擬合 GP（最大化邊際概似得到超參數），得到 $\mu_n(x),\sigma_n^2(x)$。**
    
2. 計算 $f_n^*=\max_i y_i, \; y_i=f(x_i)+\varepsilon_i$，定義 $\mathrm{EI}_n(x)$。
    
3. 求最好的下個點
$$x_{n+1}=\arg\max_{x \in \mathcal{X} }\mathrm{EI}_n(x)$$
    
4. 評估昂貴的 $f$，得 $y_{n+1}=f(x_{n+1})+\varepsilon$。
    
5. $\mathcal{D}{n+1}=\mathcal{D}n\cup{(x{n+1},y{n+1})}$，回到 1，直到用完預算。

## 2. Surrogate Model：Gaussian Process
[[Gaussian Process]]
- **(重點)Conclusion**: 下個目標值 $f(x)$根據已有目標值 $f(x_{1:n})\sim \mathcal{N}(\mu_0(x_{1:k}), \Sigma_0(x_{1:k},x_{1:k}))$ 的條件決定 $f(x)|f(x)_{1:n} \sim \mathcal{N}(\mu_n(x), \sigma_n^2(x))$
    - $\mu_n(x) = \mu_0(x) + \Sigma_0(x,x_{1:n})\Sigma_0(x_{1:n},x_{1:n})^{-1}(f(x_{1:n})-\mu_0(x_{1:n}))$
    - $\sigma_n^2=\Sigma_0(x,x')-\Sigma_0(x,x_{1:n})\Sigma_0(x_{1:n},x_{1:n})^{-1}\Sigma_0(x_{1:n},x)$
    - [[Proof of Conditional Mean & Covariance]]
### Prior

對任意 $n$ 個觀測資料 $x_{1:n}={x_1,\dots,x_n}\subset\mathbb{R}^d$：

$$f(x_{1:n})=\big(f(x_1),\dots,f(x_n)\big)^\top\sim\mathcal{N}\big(\mu_0(x_{1:n}),\ \Sigma_0(x_{1:n},x_{1:n})\big)$$

其中：

- **Mean function** $\mu_0(x_{1:n})=\big(\mu_0(x_1),\dots,\mu_0(x_n)\big)^\top\in\mathbb{R}^n$
    - 均值函數可取 $\mu_0(x)=\mu+\sum_{i=1}^p\beta_i\Phi_i(x)$，參數為 $\mu,\beta_i$（實務上常取常數 $\mu_0(x)=\mu$）。
- **Covariance Function**$\Sigma_0(x_{1:n},x_{1:n})\in\mathbb{R}^{n\times n}$，第 $(i,j)$ 項為 $\Sigma_0(x_i,x_j)$：

$$\Sigma_0(x_{1:n},x_{1:n})=\begin{bmatrix}\Sigma_0(x_1,x_1)&\cdots&\Sigma_0(x_1,x_n)\\ \vdots&\ddots&\vdots\\ \Sigma_0(x_n,x_1)&\cdots&\Sigma_0(x_n,x_n)\end{bmatrix}$$

- Covariance function（kernel）可取（參數為 $\alpha_0,\{\alpha_i\}_{i=1}^d$）：

$$\Sigma_0(x,x')=\alpha_0\exp\;\Big(-\sum_{i=1}^d\alpha_i(x_i-x_i')^2\Big)$$

$\alpha_0$ 為訊號變異數，$\alpha_i$ 控制第 $i$ 維的長度尺度（$\alpha_i$ 越大，$f$ 沿該維變化越快）。

以上合稱 GP prior：

$$f\sim\mathcal{GP}\big(\mu_0(x),\ \Sigma_0(x,x')\big)$$

- $\mu_0(x)=\mathbb{E}[f(x)]$
- $\Sigma_0(x,x')=\mathrm{Cov}\big(f(x),f(x')\big)=k(x,x')$，決定 $f$ 的平滑度假設：$x,x'$ 越近，$\Sigma_0$ 越大，$f(x)$ 與 $f(x')$ 越相關。

常用 kernel：

- RBF：即上式（平方指數型）。其樣本函數無限可微，對實際系統假設過強。
- Matérn 5/2：樣本函數二次可微，BO 實務較常用。

### Posterior
$y_i=f(x_i)+\varepsilon_i, \varepsilon_i \sim \mathcal{N}(0, \sigma_{\varepsilon}^2)$   
給定 $\mathcal{D}_n={(x_i,y_i)}_{i=1}^n$，記 $y_{1:n}=(y_1,\dots,y_n)^\top$。

**Step 1：寫出聯合分布。** 對任一待預測點 $x$：

$$\begin{bmatrix}y_{1:n}\\ f(x)\end{bmatrix}\sim\mathcal{N}\left(\begin{bmatrix}\mu_0(x_{1:n})\\ \mu_0(x)\end{bmatrix},\begin{bmatrix}\Sigma_0(x_{1:n},x_{1:n})+\sigma_\varepsilon^2I & \Sigma_0(x_{1:n},x)\\ \Sigma_0(x,x_{1:n}) & \Sigma_0(x,x)\end{bmatrix}\right)$$

**Step 2：套用多元高斯條件分布公式。** 若 $\begin{bmatrix}a\\ b\end{bmatrix}\sim\mathcal{N}\left(\begin{bmatrix}\mu_a\\\mu_b\end{bmatrix},\begin{bmatrix}\Sigma_{aa} & \Sigma_{ab} \\ \Sigma_{ba} & \Sigma_{bb}\end{bmatrix}\right)$，則

$$b\mid a\sim\mathcal{N}\big(\mu_b+C^\top A^{-1}(a-\mu_a),\ B-C^\top A^{-1}C\big)$$

**結果：**

$$f(x)\mid\mathcal{D}_n\sim\mathcal{N}\big(\mu_n(x),\sigma_n^2(x)\big)$$

$$\mu_n(x)=\mu_0(x)+\Sigma_0(x,x_{1:n})\big(\Sigma_0(x_{1:n},x_{1:n})+\sigma_\varepsilon^2I\big)^{-1}\big(y_{1:n}-\mu_0(x_{1:n})\big)$$

$$\sigma_n^2(x)=\Sigma_0(x,x)-\Sigma_0(x,x_{1:n})\big(\Sigma_0(x_{1:n},x_{1:n})+\sigma_\varepsilon^2I\big)^{-1}\Sigma_0(x_{1:n},x)$$

當觀測無雜訊（$\sigma_\varepsilon^2=0$，$y_i=f(x_i)$）時，退化為原筆記第一段的形式：

$$\mu_n(x)=\mu_0(x)+\Sigma_0(x,x_{1:n})\Sigma_0(x_{1:n},x_{1:n})^{-1}\big(f(x_{1:n})-\mu_0(x_{1:n})\big)$$

### 超參數

核函數的超參數 $\theta = (\ell, \sigma_f, \sigma_\varepsilon)$ 通常以最大化 log marginal likelihood 求得：

$$  
\log p(\mathbf{y} \mid X, \theta) = -\tfrac{1}{2}\mathbf{y}^\top K_\theta^{-1}\mathbf{y} - \tfrac{1}{2}\log|K_\theta| - \tfrac{n}{2}\log 2\pi, \quad K_\theta = K + \sigma_\varepsilon^2 I  
$$

### 計算成本

矩陣求逆的複雜度為 $O(n^3)$。

## 3. Acquisition Function

Acquisition function $\alpha(x; \mathcal{D}_n)$ 以後驗 $(\mu_n, \sigma_n)$ 量化「在 $x$ 評估的價值」，下一點取：

$$  
x_{n+1} = \arg\max_{x\in\mathcal{X}} \alpha(x; \mathcal{D}_n)  
$$

$\alpha$ 的計算成本低，可以用 L-BFGS 搭配多起點，或隨機搜尋來最佳化。

令 $f_n^*$ 為目前最佳觀測值，常見的 acquisition function 如下。

**Expected Improvement (EI)**
：**設無noise**, 超過目前最佳值得期望幅度
- $\mathbb{E}_n$ 表示以 $\mathcal{D}_n$ 為條件的期望。
$$\text{EI}_n(x) = \mathbb{E}\big[\max(f(x) - f_n^*\;,\; 0)\big]  $$
$$f(x)\mid f(x_{1:n})\sim\mathcal{N}\big(\mu_n(x),\sigma_n^2(x)\big)$$

- 常態分布的Random variable可以寫成：$f(x)=\mu_n+\sigma_nZ,\qquad Z\sim\mathcal{N}(0,1)$
- 令 $\Delta_n(x)=\mu_n(x)-f_n^*$，

$$f(x)-f_n^*=\underbrace{(\mu_n-f_n^*)}_{\Delta_n}+\sigma_nZ=\Delta_n+\sigma_nZ$$
只有 $\Delta_n+\sigma_nZ>0$，即 $Z>-\frac{\Delta_n}{\sigma_n}$ 時，被積函數才非零($Z>-\frac{\Delta_n}{\sigma_n}$積分)：


$$\mathrm{EI}_n=\int_{-\frac{\Delta_n}{\sigma_n}}^{\infty}(\Delta_n+\sigma_nz)\varphi(z)dz$$

拆成兩個積分

$$\mathrm{EI}_n=\underbrace{\Delta_n\int_{-\frac{\Delta_n}{\sigma_n}}^\infty\varphi(z)dz}_{\text{ A}}+\underbrace{\sigma_n\int_{-\frac{\Delta_n}{\sigma_n}}^\infty z\;\varphi(z)dz}_{\text{ B}}$$
- Part **A**: $\int_a^\infty\varphi(z)dz$ 是 $P(Z>a)$
$$P(Z>-\frac{\Delta_n}{\sigma_n})=1-\Phi(-\frac{\Delta_n}{\sigma_n})=\Phi(\frac{\Delta_n}{\sigma_n})$$
$$ \rightarrow \text{A}:\Delta_n\Phi(\frac{\Delta_n}{\sigma_n})$$
- Part **B**: 
	- Standard Normal Distribution [[Standard Normal Distribution]]
		- pdf: $\varphi(z)=\frac{1}{\sqrt{2\pi}}e^{-z^2/2}$
		- cdf: $\Phi(z) = P(z≥Z)$ 
$$\int_{-\frac{\Delta_n}{\sigma_n}}^\infty z\;\varphi(z)dz=\big[-\varphi(z)\big]_{-\frac{\Delta_n}{\sigma_n}}^\infty=0-(-\varphi(-\frac{\Delta_n}{\sigma_n}))$$
$$=-\varphi(\frac{\Delta_n}{\sigma_n})=\varphi(\frac{\Delta_n}{\sigma_n})$$
$$\rightarrow \mathbf{B}:\sigma_n\varphi(\frac{\Delta_n}{\sigma_n})$$
so: $\mathrm{EI}_n=\mathbf{A} + \mathbf{B}$
$$\mathrm{EI}_n(x)=\Delta_n(x)\Phi\Big(\frac{\Delta_n(x)}{\sigma_n(x)}\Big)+\sigma_n(x)\varphi\Big(\frac{\Delta_n(x)}{\sigma_n(x)}\Big)$$

- 第一項在 $\mu_n$ 高時大，對應 exploitation。($\Delta_n(x)=\mu_n(x)-f_n^*$)
- 第二項在 $\sigma_n$ 高時大，對應 exploration。

then 找下個點 $x$ 
$$x_{n+1}=\arg\max_{x\in\mathcal{X}}\ \mathrm{EI}_n(x)$$



**Probability of Improvement (PI)**

$$  
\text{PI}(x) = \Phi\left(\frac{\mu_n(x) - f_n^* - \xi}{\sigma_n(x)}\right)  
$$

只考慮改善的機率，不考慮改善幅度，因此傾向過度 exploitation。



**GP-UCB**

$$  
\text{UCB}(x) = \mu_n(x) + \beta_n^{1/2},\sigma_n(x)  
$$

在適當的 $\beta_n$ 排程下有 sublinear cumulative regret 的理論保證（Srinivas et al., 2010）。

**其他**

- **Thompson Sampling**：從後驗抽一條樣本函數 $\tilde{f} \sim p(f\mid\mathcal{D}_n)$，取 $\arg\max \tilde{f}$。
- **資訊理論類**（Entropy Search、PES、MES）：選擇最能降低 $x^*$ 或 $f(x^*)$ 後驗熵的點。
- **Knowledge Gradient**：
	- 最大化評估一次後 $\max_x \mu_{n+1}(x)$ 的期望提升，再有noise下最好
	- computational **inefficient**，沒有close form, 只能蒙特卡洛
$$\mu^*_{n+1}=\max_{x'}\mu_{n+1}(x')$$
$$\text{KG}_n(x)=E_n(\mu^*_{n+1}-\mu^*_n\;|\;x_{n+1}=x)$$
$$x_{n+1}=arg\max_x\text{KG}_n(x)$$

## 4. 演算法流程


輸入：搜尋空間 X，預算 N，初始點數 n0
1. 以 Latin Hypercube / Sobol 取 n0 個初始點，評估得 $D_{n0}$
2. for n = n0, ..., N-1:
3. 以 $D_n$ 擬合 GP（更新超參數 θ）
4. $x_{n+1} = argmax_x α(x; D_n)$
5. 評估$y_{n+1} = f(x_{n+1}) + ε$
6. $D_{n+1} = D_n ∪ {(x_{n+1}, y_{n+1})}$
7. 回傳 $argmax_{x_i ∈ D_N} y_i$（或 $argmax \;μ_N(x)$）


## 5. 限制與對應變體

|問題|原因|常見解法|
|---|---|---|
|高維（$d > 20$）|GP 在高維的建模能力差，且 $\alpha$ 難以最佳化|TuRBO（local trust region）、REMBO/ALEBO（隨機低維嵌入）、加性核|
|樣本數大|$O(n^3)$|Sparse GP、以 random forest（SMAC）或神經網路作 surrogate|
|類別或整數參數|標準核假設連續空間|SMAC、TPE、專用核（如 CoCaBO）|
|平行評估|一次只提出一點|Batch BO：q-EI、local penalization|
|有限制條件|部分 $x$ 不可行|Constrained BO：$\text{EI}(x)\cdot \Pr[c(x)\le 0]$|
|多目標|需權衡多個指標|EHVI（Expected Hypervolume Improvement）|
|評估成本可分級|可先用便宜的近似評估|Multi-fidelity BO：Hyperband 結合 BO（BOHB）|

**TPE**（Tree-structured Parzen Estimator，Hyperopt/Optuna 預設）也屬於 BO 家族。差別在於 GP 建模的是 $p(y\mid x)$，TPE 改為建模 $p(x \mid y)$：以分位數 $y^*$ 將資料切成好、壞兩組，分別以 KDE 估計 $l(x)$、$g(x)$，並最大化 $l(x)/g(x)$。這在數學上等價於最大化 EI。

## 6. 在 DB Knob Tuning 中的角色

資料庫參數調校符合 BO 的所有前提：

- 每組 knob 設定需要實際跑一次 workload 才能量測 throughput 或 latency，評估昂貴。
- knob 與效能之間沒有解析關係，屬於黑箱。
- 量測結果受系統狀態影響，有雜訊。

因此 BO 是這個領域的主流 baseline：

- **iTuned**（VLDB 2009）：GP 搭配 EI。
- **OtterTune**（SIGMOD 2017）：GP-based BO，並以歷史 workload 的資料 warm-start。
- **ResTune**：constrained BO，在滿足 SLA 限制下最佳化資源使用。

報告 DOT 時，若該論文有和這類方法比較，重點在於 BO 在 knob tuning 的已知瓶頸：

- knob 數量多，屬於高維問題。
- knob 含類別型與整數型。
- 需要大量實際 workload 執行，冷啟動成本高。

## 參考文獻

- Frazier, _A Tutorial on Bayesian Optimization_, arXiv:1807.02811, 2018
- Shahriari et al., _Taking the Human Out of the Loop: A Review of Bayesian Optimization_, Proc. IEEE, 2016
- Rasmussen & Williams, _Gaussian Processes for Machine Learning_, MIT Press, 2006



| BO                      | ML                   |
| ----------------------- | -------------------- |
| 擬合 GP                   | 訓練模型（要最小化 loss，擬和資料） |
| $\mathcal{D}_n$: 少量資料   | 訓練集，大量資料             |
| 選最好路徑 $\in \mathcal{X}$ | 預測結果                 |


