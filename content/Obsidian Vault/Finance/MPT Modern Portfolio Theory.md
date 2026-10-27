：組合投資風險取決於資產配置與分散（資產間共變異結構）


given $N$ 支資產、報酬 $R \in \mathbb{R}^n$ 
- 期望報酬 $\mu = E(R)$
- 共變異數矩陣 $\Sigma = Cov(R)$ 
- 投資組合權重 $w^TR$ 
$$E(R_p) = w^T\mu, \; Var(R_p) = w^T\Sigma w$$
ex.兩資產case [[Joint Distribution]]
$$\mu_p = w_1\mu_1 + w_2\mu_2$$
$$\sigma_p^2 = w_1^2\sigma_1^2 + w_1^2\sigma_2^2 + 2w_1w_2\sigma_{12}$$
風險 $\sigma$ 不是線性，由 $\rho_{12}$ 決定($2w_1w_2\sigma{12} = 2w_1w_2\rho_{12}\sigma_1\sigma_2$ )

ex.取 σ₁ = σ₂ = 20%，w₁ = w₂ = 0.5，加權平均標準差 = 20%：

| ρ₁₂ | σ_p² | σ_p   |
| --- | ---- | ----- |
| 1   | 0.04 | 20%   |
| 0.5 | 0.03 | 17.3% |
| 0   | 0.02 | 14.1% |
| −1  | 0    | 0%    |

只要 ρ₁₂ < 1，σ_p 就小於 20%。兩資產漲跌不完全同步時，部分波動會互相抵銷。


### Efficient Frontier
：在不同波動程度下，最有效的投資組合。
給定目標報酬 $\mu_p$ , 求最小化風險的投資權重 $w$ 
$$\min_w \ \tfrac{1}{2} w^\top \Sigma w \quad \text{s.t.} \quad w^\top \mu = \mu_p,\ \ \mathbf{1}^\top w = 1$$
used Lagrange Multiplier $\frac{\partial}{\partial w}\mathcal{L} = 0$ 
$$\frac{\partial}{\partial w} =\tfrac{1}{2} w^\top \Sigma w- \lambda(w^\top \mu - \mu_p)-v(\mathbf{1}^\top w-1)= 0$$
$$\Sigma w = \lambda\mu + v\mathbf{1} \Rightarrow w^* = \Sigma^{-1}(\lambda\mu + v\mathbf{1})$$
帶回限制式
$$\mu^\top w^* = \lambda,\mu^\top\Sigma^{-1}\mu + v,\mu^\top\Sigma^{-1}\mathbf{1} = \mu_p$$

$$\mathbf{1}^\top w^* = \lambda,\mathbf{1}^\top\Sigma^{-1}\mu + v,\mathbf{1}^\top\Sigma^{-1}\mathbf{1} = 1$$
所以
- $A = \mathbf{1}^\top\Sigma^{-1}\mathbf{1}$
- $B = \mathbf{1}^\top\Sigma^{-1}\mu$
- $C = \mu^\top\Sigma^{-1}\mu$

$$\begin{bmatrix} C & B \\ B & A \end{bmatrix}\begin{bmatrix}\lambda\\ v\end{bmatrix} = \begin{bmatrix}\mu_p\\ 1\end{bmatrix}$$

$$\begin{bmatrix}\lambda\\ v\end{bmatrix} = \frac{1}{AC - B^2}\begin{bmatrix} A & -B \\ -B & C \end{bmatrix}\begin{bmatrix}\mu_p\\ 1\end{bmatrix}$$
$$\lambda = \dfrac{A\mu_p - B}{D}\quad v = \dfrac{C - B\mu_p}{D}$$
$$Var(R_p) = w^T\Sigma w = w^T(\lambda\mu + v\mathbf{1}) $$
$$=\lambda \mu^tw + v\mathbf{1}^Tw = \lambda\mu_p+v$$
$$\sigma_p^2 = \frac{A\mu_p^2 - 2B\mu_p + C}{AC - B^2}$$
是 $(\sigma_p, \mu_p)$ 平面上的雙曲線（$(\sigma^2_p, \mu_p)$ 平面上的拋物線）
![[Pasted image 20260920154213.png]]