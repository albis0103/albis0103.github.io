
$X \sim \mathcal{N}(\mu, \sigma^2)$ 
$$f_X(x)=\frac{1}{\sqrt{2\pi}\sigma}e^{-\frac{(x-\mu)^2}{2\sigma^2}}$$
$$M_X(t)=e^{\mu t}\frac{1}{2}\sigma^2t^2$$
$Y = X + c$ or $X-c$
- $Y \sim \mathcal{N}(\mu±c, \sigma^2)$
$Y = X*c$ or $\frac{X}{c}$ 
- $Y \sim \mathcal{N}(c\mu,c^2\sigma^2)$ or $Y \sim \mathcal{N}(\frac{\mu}{c},\frac{c^2}{\sigma^2})$

1. $\bar X_n \sim N(\mu, \frac{\sigma^2}{n})$ and $\frac{\sqrt n (\bar X - \mu)}{\sigma} ~ N(0, 1)$ 
2. The random variable $\bar X_n$  and the random vector $\mathbf{Y}=(Y_1,\cdots,Y_n)$ are independent
	- $Y_i = X_i-\bar X_n$ 
3. $\bar X_n$ and $S_n^2$ are independent
	- $S_n^2=\frac{1}{n-1}\sum (X_i-\bar X_i)^2$ is  $\mathbf{Y}$ Transformation
4. $\frac{(n-1)S^2}{\sigma^2} \sim \chi^2(n-1)$  is chi-square distribution
5. $\frac{\sqrt n (\bar X_n - \mu)}{S_n} \sim t_{n-1}$
	- $\frac{\sqrt n (\bar X_n - \mu)}{S_n}=\frac{\sqrt n (\bar X_n - \mu)\sigma}{\sqrt \frac{S_n^2(n-1)}{\sigma^2}\frac{1}{n-1}}$
		- $Z = \frac{\sqrt{n}(\bar X_n - \mu)}{\sigma} \sim N(0,1)$
		- $V = \frac{(n-1)S_n^2}{\sigma^2} \sim \chi^2_{n-1}$ 
	- then $T = \frac{Z}{\sqrt{V/(n-1)}} \sim t_{n-1}.$
