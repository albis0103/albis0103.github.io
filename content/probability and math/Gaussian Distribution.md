
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



### Bivariate Gaussian Distrubution
: Describe Two Random Variable $X = \begin{bmatrix} X_1 \\ X_2 \end{bmatrix}$ Joint Distribution
- Mean vector: $\boldsymbol{\mu} = \begin{bmatrix} \mu_1 \\ \mu_2 \end{bmatrix}$
- Covariance Matrix: $\boldsymbol{\Sigma} = \begin{bmatrix} \sigma_1^2 & \rho\sigma_1\sigma_2 \\ \rho\sigma_1\sigma_2 & \sigma_2^2 \end{bmatrix}, \quad \sqrt{det(\boldsymbol{\Sigma})} = \sqrt{|\boldsymbol{\Sigma}|} =\sqrt{\sigma^2_1\sigma^2_2-\rho^2 \sigma^2_1\sigma^2_2}=\sqrt{1-\rho^2}\sigma_1\sigma_2$ 
$$f(x_1, x_2) = \frac{1}{2\pi \sigma_1 \sigma_2 \sqrt{1-\rho^2}} \exp\left( -\frac{1}{2(1-\rho^2)} \left[ \frac{(x_1-\mu_1)^2}{\sigma_1^2} - \frac{2\rho(x_1-\mu_1)(x_2-\mu_2)}{\sigma_1\sigma_2} + \frac{(x_2-\mu_2)^2}{\sigma_2^2} \right] \right)$$


### Multivariate Gaussian Distribution
: $\mathbf{X} = [X_1, X_2, \dots, X_D]^T \sim \mathcal{N}(\boldsymbol{\mu}, \boldsymbol{\Sigma})$
- Mean vector: $\boldsymbol{\mu} = \mathbb{E}[\mathbf{X}] = \begin{bmatrix} \mu_1 \\ \mu_2 \\ \vdots \\ \mu_D \end{bmatrix}\in \mathbb{R}^D$
- Covariance Matrix: $\boldsymbol{\Sigma} = \mathbb{E}[(\mathbf{X} - \boldsymbol{\mu})(\mathbf{X} - \boldsymbol{\mu})^T] \in\mathbb{R}^{D \times D}$
	- $\boldsymbol{\Sigma}_{ij} = \text{Cov}(X_i, X_j)$
	- $\boldsymbol{\Sigma}$ is Symmetric matrix and Positive Difinite

$$f(\mathbf{x})=\frac{1}{(2\pi)^{\frac{1}{D}}|\boldsymbol{\Sigma}|^{\frac{1}{2}}}exp\left(-\frac{1}{2}(\mathbf{x}-\mathbf{\mu})^T\boldsymbol{\Sigma}^{-1}(\mathbf{x}-\mathbf{\mu})\right)$$

- $\boldsymbol{\Sigma}^{-1}$: Inverse Covariance Matrix, equal **Precision Matrix** $\mathbf{\Lambda}$
- **Marginal Distribution** is Gaussian Distribution $\mathbf{X}_A \sim \mathcal{N}(\boldsymbol{\mu}_A, \boldsymbol{\Sigma}_{AA})$ 
$$p(\mathbf{X}_A) = \int p(\mathbf{X}_A, \mathbf{X}_B) \, d\mathbf{X}_B$$
- Conditional Distribution is Gaussian Distribution $\mathbf{X}_A \mid \mathbf{X}_B = \mathbf{x}_B$
	- Conditional Mean
$$\boldsymbol{\mu}_{A\vert{}B} = \boldsymbol{\mu}_A + \boldsymbol{\Sigma}_{AB} \boldsymbol{\Sigma}_{BB}^{-1} (\mathbf{x}_B - \boldsymbol{\mu}_B)$$
	- Conditional Covariance
$$\boldsymbol{\Sigma}_{A\vert{}B} = \boldsymbol{\Sigma}_{AA} - \boldsymbol{\Sigma}_{AB} \boldsymbol{\Sigma}_{BB}^{-1} \boldsymbol{\Sigma}_{BA}$$
[[Proof of Conditional Mean & Covariance]]