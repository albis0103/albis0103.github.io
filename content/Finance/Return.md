- 報酬率 ex.每年長高幾公分
	- 平穩(Stationary)
- 價格 ex.身高會一直長高
	- 非平穩(Non - Stationary)
model都用return, 因為$\mu, \sigma$ 必須是穩定

- 簡單報酬 simplereturn
$$r_t^s = \frac{P_t-P_{t-1}}{P_{t-1}}$$
- 對數報酬 logreturn
$$r_t^l = log(\frac{P_t}{P_{t-1}})=logP_t-logP_{t-1}=ln(r_t^s)$$
note: $ln(1 + x) \approx x, x\rightarrow 0$ [[Taylor Expansion]]
$$lnP_t-lnP_{t-1}=ln(1+\frac{P_t-P_{t-1}}{P_{t-1}})=ln(1+r_t^s)$$
