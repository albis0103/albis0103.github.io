
$$k^{th}\; moment: \mu_k = E[X^k]=\int x^k dF_X(x)=\begin{cases}\int_{-\infty}^{\infty} x^k f(x)\,dx \\ \sum_x x^k p(x)\end{cases}$$

can use $k^{th}$ moments to calculate mean, var, skewness, kurtosis, **central moment**
- $k=1$：$\mu_1 = E[X-\mu] = 0$
- $k=2$：$\mu_2 = E[(X-\mu)^2] = \mathrm{Var}(X)$
- $k=3$：$\mu_3 = E[(X-\mu)^3]$ **Skewness** $=\frac{\mu'_3}{\sigma^3}$
- $k=4$：$\mu_4 = E[(X-\mu)^4]$, **Kurtosis** $=\frac{\mu_4'}{\sigma^4}$ 

**Central moment**
$$k^{th}\;central\; memont=\mu_k' =E[(X-\mu)^k]$$

