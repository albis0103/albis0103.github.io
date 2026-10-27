
Standard linear regression [[RLS Regularized Least Squares]]
$$J(w)=\frac{1}{2}\sum_{b=1}^N(w^T\phi(x_b)-t_b)^2+\frac{\lambda}{2}w^Tw$$
- $\phi \in \mathbb{R}^M$：樣本$\mathbf{x}$ 的特徵向量  map $x_i \rightarrow$ 特徵空間 ex.[[Gaussian Kernel]]
$$

\boldsymbol{\phi}(\mathbf{x})=\begin{bmatrix}\phi_0(\mathbf{x})\\ \phi_1(\mathbf{x})\\ \vdots\\ \phi_{M-1}(\mathbf{x})\end{bmatrix}\in\mathbb{R}^{M}

$$
- $t_i$ ：目標值
- $\lambda$：正則化係數
$$

\boldsymbol{\Phi}=\begin{bmatrix}\boldsymbol{\phi}(\mathbf{x}_1)^\top\\ \boldsymbol{\phi}(\mathbf{x}_2)^\top\\ \vdots\\ \boldsymbol{\phi}(\mathbf{x}_N)^\top\end{bmatrix}\in\mathbb{R}^{N\times M},\qquad \Phi_{nj}=\phi_j(\mathbf{x}_n)

$$
let $\Delta_wJ = 0$

$$\mathbf{w}=\sum_{n}\underbrace{\left(-\frac1\lambda\big(\mathbf{w}^\top\boldsymbol{\phi}(\mathbf{x}_n)-t_n\big)\right)}_{a_n}\boldsymbol{\phi}(\mathbf{x}_n)=\boldsymbol{\Phi}^\top\mathbf{a}$$

let $\mathbf{w} = \Phi^Ta$ , $J(w) \rightarrow J(a)$ 
![[截圖 2026-10-01 10.40.35.jpeg|400]]

$$a_n=-\frac1\lambda(\mathbf{w}^T\phi(\mathbf{x}_n))-t_n)\;\rightarrow\;\lambda a_n ==-(\mathbf{w}^T\phi(\mathbf{x}_n))-t_n)$$
$$\lambda\mathbf{a}=-(\Phi \mathbf{w} - \mathbf{t})$$
and $\mathbf{w} = \Phi^Ta$
$$\Phi \Phi^Ta+\lambda\mathbf{a}=t$$
let Gram metrix $\mathbf{K} = \Phi\Phi^T$ 
$$(\mathbf{K}+\lambda I)a=t$$
so:
$$a=(\mathbf{K}+\lambda I)^{-1}-t$$