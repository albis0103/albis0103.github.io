
$$\theta_t \leftarrow \theta_{t-1}-\eta\frac{\hat{m_t}}{\sqrt{\hat{v_t}}}$$

evolution by **RMSProp**:$\theta_t \leftarrow \theta_{t-1}-\eta\frac{g_t}{\sqrt{\hat{v_t}}}$

Momentum: simulate the physical activity
- $\hat{v_t}=\frac{v_t}{1-\alpha}, v_t=\alpha v_{t-1}+(1-\alpha)g_t^2$
- $\hat{m_t}=\frac{m_t}{1-\beta}, m_t=\alpha m_{t-1}+(1-\beta)g_t$
[[牛頓第二定律]] $F=\frac{d(mv)}{dt}$
