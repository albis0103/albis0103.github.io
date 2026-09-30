![[Pasted image 20260902173504.png]]
PINN: 將物理規律(PDE)作為正則項，目標用神經網路逼近PDE($u_\theta \rightarrow u$)
$$u_t+\mathcal{N}[u;\lambda]=0\;,x\in \Omega, t\in [0,T]$$
- $u(x,t)$: 目標，PDE解
- $u_\theta(x,t)$ : model output

AD(Auto-Differentiation)：對網路輸出往前計算$x,t$偏導數, 設$\frac{\partial u_{\theta}}{\partial a^{L-1}}=1$
- forward: $x,t\rightarrow z^{(1)}\rightarrow a^{(2)} \rightarrow \cdots \rightarrow \hat u_{\theta}$ 
- backward: 
	- $\frac{\partial u_\theta}{\partial a^{(L-1)}} = 1\cdot\frac{\partial a^{(L)}}{\partial a^{(L-1)}},\quad \frac{\partial u_\theta}{\partial a^{(L-2)}} = \frac{\partial u_\theta}{\partial a^{(L-1)}}\cdot\frac{\partial a^{(L-1)}}{\partial a^{(L-2)}},\ \ldots \dfrac{\partial u_\theta}{\partial x}$ 
	- $\frac{\partial u_\theta}{\partial a^{(L-1)}} = 1\cdot\frac{\partial a^{(L)}}{\partial a^{(L-1)}},\quad \frac{\partial u_\theta}{\partial a^{(L-2)}} = \frac{\partial u_\theta}{\partial a^{(L-1)}}\cdot\frac{\partial a^{(L-1)}}{\partial a^{(L-2)}},\ \ldots \dfrac{\partial u_\theta}{\partial t}$ 
Residual $f_{\theta}$ : PDE - left side，用Auto - Differentiation算物理約束的Loss $\mathcal{L}_{pde}$ 
$$f_{\theta}(x,y) = u_t+\mathcal{N}[u;\lambda]$$

Loss function
$$\mathcal{L}(\theta) = \mathcal{L}_{data}(\theta) + \lambda_{PDE}\mathcal{L}_{PDE}(\theta) + \lambda_{BC}\mathcal{L}_{BC}(\theta)$$
$$

\mathcal{L}_{data}(\theta) = \frac{1}{N_d}\sum_{i=1}^{N_d} \left| u_\theta(x_d^i, t_d^i) - u^i \right|^2

$$

$$

\mathcal{L}_{pde}(\theta) = \frac{1}{N_f}\sum_{i=1}^{N_f} \left| f_\theta(x_f^i, t_f^i) \right|^2

$$

$$

\mathcal{L}_{bc}(\theta) = \frac{1}{N_b}\sum_{i=1}^{N_b} \left| \mathcal{B}[u_\theta](x_b^i, t_b^i) \right|^2

$$
