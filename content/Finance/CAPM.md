每個期望報酬由$\beta$ 決定
$$R_i = R_f + \beta_i(R_m-R_f)$$
- $\beta$: 風險係數 $R_i \; v.s\; R_f$
$$\beta_i = \frac{R_i-R_f}{R_m-R_f}=\begin{cases}>1, & R_i>R_m\\=1, &R_i=R_m\\<1, & R_i<R_m\end{cases}$$
$$\beta_i = \frac{\sigma_{im}}{\sigma_m^2}=\rho_{im}\frac{\sigma_i}{\sigma_m}$$
Proof: 

$$R = \alpha + \beta F + \epsilon$$

$R$、$F$ 是觀察得到的，α、β、ε 是未知的。  
$$\epsilon = R - \alpha - \beta F$$
兩個條件：

- **(C1)** $\mathbb E[\epsilon] = 0$：ε 沒有系統性的偏差，由 $\alpha + \beta,\mathbb E[F]$ 完整描述。
- **(C2)** $\mathrm{Cov}(F,\epsilon) = 0$：ε 與 $F$ 不相關。


**對模型取期望，用 (C1)：**  
$$\mathbb E[R] = \alpha + \beta,\mathbb E[F] + \underbrace{\mathbb E[\epsilon]}_{0} \ \Longrightarrow\ \alpha = \mathbb E[R] - \beta,\mathbb E[F]$$

**對模型兩邊與 $F$ 取共變異數，用 (P2) 與 (C2)：**  
$$\text{Cov}(R,R) = \text{Cov}(\alpha,F) +\text{Cov}(\beta F,F)+\text{Cov}(\epsilon F)$$

$$\mathrm{Cov}(R,F) = \underbrace{\mathrm{Cov}(\alpha,F)}_{0} + \beta,\mathrm{Var}(F) + \underbrace{\mathrm{Cov}(\epsilon,F)}_{0} \ \Longrightarrow\ \beta = \frac{\mathrm{Cov}(R,F)}{\mathrm{Var}(F)}$$



**CPL Capital Market Line**
![[Pasted image 20260917232554.png]]

[[MPT Modern Portfolio Theory]]
