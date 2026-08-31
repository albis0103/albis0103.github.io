When $p(X)\sim Unifrom$ 可得 MAX entropy

pf:

$H(p)=E_{X\sim P}[ln(p(x))]$
$= -\sum p(x)lnp(x)$ (discrete)
$=-\int p(x)lnp(x)dx$ (continuous)

By Lagrange Multiplier,

given condition $\int p(x)dx = 1, \int p(x)dx-1=0$

$max_pL[p]=-\int p(x)lnp(x)dx - \lambda (\int p(x)dx -1)= \int (-p_xlnp_x - \lambda p_x )dx-\lambda$ 
$\frac{\partial L}{\partial p_x}=0, \int(-\frac{\partial p_xlnp_x}{\partial p_x}-\frac{\partial \lambda p_x}{\partial p_x})dx-\frac{\partial \lambda}{\partial p_x}=0$
$\int(-lnp_x+\frac{p_x}{p_x}-\lambda)dx-0=0$ 
$-lnp_x+1-\lambda-0=0$
$p(x)=e^{-\lambda-1}$, 與x無關
