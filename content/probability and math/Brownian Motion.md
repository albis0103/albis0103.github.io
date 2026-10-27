標準布朗運動(Brownian Motion, $B_t$)（或稱Wiener Process, $W_t$)
：用於描述市場隨機震盪與雜訊。ex. Vasicek, Black - Scholes 模型

物理學中：花粉微粒在水中受大量水分子隨機碰撞的不規則運動
金融市場：把水分子想像成市場上隨時爆發的海量資訊
![[IMG_3279.jpg|459]]
四大特徵
1. Standard Initialization: at time $t_0$ , $W_0=0$ 
2. Independent Increments
	- $\forall \; 0 ≤t_1<t_2<\cdots< t_n$ 
	- $W_{t_2}-W_{t_1}, W_{t_3}-W_{t_2}, \cdots, W_{t_n} - W_{t_{n-1}}$ mutual independent
3. Stationary Gaussian Increments
	- for any $0 ≤ s < t$
	- $W_t-W_s \sim \mathcal{N}(0, t-s)$ 
	- note: observe interval $dt$ , the variance of $dW_t$ Increment is $dt$ , so that $(dW)^2=dt$ 
4. Continuous but Non-diffferentiable
	- so we need Stochastic Calculus
Because $\Delta W_t \sim \mathcal{N}(0, \Delta t)$  [[Ito Multiplication Table]]
- $(dt)^2=0$
- $dt\cdot dW = 0$
- $(dW)^2=\Delta t$ 

**GBM**(Geometric Brownian Motion)
:GBM $S_t$ that satisfy the **SDE** process
- Trend: Long-term motion
- Shock: System Random Noise
![[IMG_3280.jpg|516]]
$$dS_t = \mu S_tdt + \sigma S_tdW_t$$
- $\mu \in \mathbb{R}$ (drift rate): 瞬時期望報酬率
- $\sigma > 0$(volatility): 波動率
- $W_t$ : Standard Brownian Motion

$$dS_t = \mu S_tdt+\sigma S_tdW_t$$

$It\hat o$'s Lemma