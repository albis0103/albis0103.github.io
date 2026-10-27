$$p_t = E_t(M_{t+1}X_{t+1})$$
- $p_t$：資產在 $t$ 期價格
- $X_{t+1}$：資產在$t+1$期payoff（收益）
	- 股票是$p_{t+1}+d_{t+1}$（股利）
- $M_{t+1}$：SDF，是隨機變數，隨$t+1$期狀態改變

ex.報酬率 $R_{t+1} = \frac{X_{t+1}}{p_t}$：資產從$t \rightarrow t+1$的總報酬(gross return)
$$E(m_{t+1}R_{t+1})=1$$




**1. 無風險利率**：無風險資產的 $R^f_{t+1}$ 在 $t$ 時點已知

$$R^f_t = \frac{1}{E_t[m_{t+1}]} = 1+r_{f,t+1}$$
- $R_t^f$: $t$ 時已知的利率
- $1+r_{f,t+1}$：$t \rightarrow t+1$間的無風險毛利率

note: discount factor：未來一單位利率$r$ , $T$ 期換算成現值
$$p_t = d\cdot x_{t+1},\quad d = \frac{1}{(1+r)^T}$$


**2. 風險溢酬由與 SDF 的共變異數決定**：[[Proof of Risk-Premium by SDF Covariance]]

$$E[r_i] - r_f = -(1+r_f)\text{Cov}(m_{t+1}r_{i,t+1})$$


資產報酬與 $m$ 負相關（在 $m$ 高的壞狀態下報酬低），則要求較高的期望報酬。

**3. 超額報酬**：零成本投資組合（價格為 0）滿足 $E_t[m_{t+1}(R^i_{t+1} - R^f_t)] = 0$。

## SDF 的具體形式

- **消費基礎模型**：$m_{t+1} = \beta,\dfrac{u'(c_{t+1})}{u'(c_t)}$，此時上式即為跨期消費的 Euler equation。
- **存在性**：Law of one price 保證存在某個 $m$ 滿足此式；無套利進一步保證存在 $m > 0$；市場完全時 $m$ 唯一。
- CAPM、APT、Fama-French 等因子模型可視為對 $m$ 設定線性形式 $m = a + b'f$ 的特例。