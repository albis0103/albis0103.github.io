 - Discrete
	- Joint PMF
		$P(X=x_0, Y = y_0)=p_{X, Y}(x_0, y_0)$ 
		- Marginal PMF: $p_X(x) = \sum_Yp_{X,Y}(x,y)P_Y(y)=\sum_XP_{X,Y}(x,y)$ 
	- Joint CDF:$F_{X,Y}(x_0, y_0)=P(X≤x_0, Y≤y_0)$ 
- Continue
	- Joint PDF $(x,y) \in \omega \in \Omega$
		$P(\omega)=\int\int_{(x,y)\in omega}f_{X,Y}(x,y)dxdy$
		- Marginal PDF:$f_X(x)=\int_{-\infty}^{\infty}f_{X,Y}(x,y)dyf_Y(y)=\int_{-\infty}^{\infty}f_{X,Y}(x,y)dx$  

Random Vector
: $n$ random variables, $X = (X_1, \cdots , X_n)^T$ 

Transformation
- Method of Event
	- Distribution $Z = X+Y, \; f_Z = \int f_X(x)f_Y(z-x)dx$ 
		- Generization $Z = X_1+\cdots + X_n$ 
			- $p_Z(z) = p_{S_n}(z),\;S_2 = X_1+X_2, \; k=1,..,n,\;$ 
			- $f_{S_k}(s) = \int f_{S_{k-1}}(x)f_{X_k}(s-x)dx$  
- Method of CDF
- Method of PDF
- Method of mgf


Condition Joint Distribution
- Discrete
	$P(X=x|Y=y)=p_{X|Y}(x|y)=\frac{p_{X,Y}(x,y)}{p_Y(y)}$
- Continue
	$P(x<X≤x+\Delta x|y<Y≤y+\Delta y)=\frac{f_{X,Y}(x,y)dxdy}{f_Y(y)dy}$ 

**Conditional Expectation**
[[Conditional Expectation]]
$h:\mathbb{R}^m \rightarrow \mathbb{R}^1$
$$E(h(Y)|X=x)=\sum_{y\in \mathcal{Y}}h(y)p_{Y|X}(y|x)$$
$$E(h(Y)|X=x)=\int_{y\in \mathcal{Y}}h(y)f_{Y|X}(y|x)$$
- $X, Y$ independent: $E_{Y|X}[h(Y)|X=x]=E_Y[h(Y)]$ 
- $E[h(X)|X=x]=h(x)$

**Law of total Expectation**: 
- $E_X\{E_{Y|X}[h(y)|X]\}=E_Y[h(x)]$
- $E_X\{E_{Y|X}[Y_i|X]\}=E_Y[Y_i]$
- Generalize: $E_{XY}[h(y)]=E_X[E_{Y|X}[h(y)]]$

**Law of total Variation**(Variation Decomposition, note:ANOVA: $SST=SSR+SSE$)[[ANOVA]]
$$Var_Y(Y_i)=Var_X[E_{Y|X}(Y_i|X)]+E_X[Var_{Y|X}(Y_i|X)]$$
so 
- $Var_Y(Y_i) ≥ Var_X[E_{Y|X}(Y_i|X)]$
	- Special case: $Var_Y=Var_XE_{Y|X}$
		- if $E_XVar_{Y|X}=0$ 
- $Var_Y(Y_i) ≥ E_X[Var_{Y|X}(Y_i|X)]$
	- Special case: $Var_Y=E_XVar_{Y|X}$
		- $Var[E(Y|X)]=0, E(Y|X)$ is constant over $x$, and $E_YE_{Y|X}=\mu_Y$ 
![[07_Expectation_wHWN 5.jpeg]]
Independent Joint Distribution

$$X_1, X_2, \cdots, X_n \text{ are indepent} \iff \begin{cases}f(X_1, \cdots , X_n)=f(X_1)f(X_2)\cdots f(X_n)\\F(X_1, \cdots , X_n)=F(X_1)F(X_2)\cdots F(X_n)\end{cases}$$

**Expectation**
:Expectation under the Joint Distribution($\mathbf{X}:\mathbf{R}^n, \mathbf{Y}:\mathbf{R}^1$ Univariate)
	Can use $\mathbf{X}$ calculate $\mathbf{Y}\;pmf$ 
	$\mathbf{X}=(X_1, \cdots, X_n), Y=g(X_1, \cdots, X_n), g:\mathbf{R}^n \rightarrow \mathbf{R}^1$
$$E[Y] = \int_{-\infty}^{\infty}ydF_Y(y)=\begin{cases}\sum_{y \in \mathcal{Y}}y\;p_Y(y), & \text{if discrete case:}F_Y(y)-F_Y(y-)=\Delta F_Y(y)\\\int_{-\infty}^{\infty}y\;f_Y(y)dy, & \text{if continue cases:}\frac{dF_Y(y)}{dy}=f_Y(y) \end{cases}$$
[[Riemann-Stieltjes Integral]]
when $\mathbf{X}$ is
- Discrete( Converges absolutely: $\sum|g(x)|p_X(x)< -\infty$)
	$E[Y]=\sum_{y \in \mathcal{Y}}y\;p_Y(y)$ 
		$=\sum_{\mathbf{x}=(x_1, \cdots, x_n)\in \mathcal{X}} g(x_1, \cdots, x_n)p_X(x_1, \cdots, x_n)$ 
- Continue(Converges absolutely: $\int |g(x)|f_X(x)dx<-\infty$)
	$E[Y]=\int_{-\infty}^{\infty}y\;f_Y(y)dy$
		$=\int_{-\infty}^{\infty}\cdots\int_{-\infty}^{\infty}g(x_1, \cdots, x_n)f_X(x_1, \cdots, x_n)dx_1 \cdots dx_n$
when $\mathbf{X}$ is Continue and $\mathbf{Y}$ is Discrete
	$E[Y]=\sum_{y \in \mathcal{Y}}y\;p_Y(y)$
		=$=\int_{-\infty}^{\infty}\cdots\int_{-\infty}^{\infty}g(x_1, \cdots, x_n)f_X(x_1, \cdots, x_n)dx_1 \cdots dx_n$
Mean of sum:$E[\sum a_iX_i]=\sum a_iE[X_i]$ 
	$E[a_0+a_1X_1+\cdots+a_nX_n]=a_0+a_1E[X_1]+\cdots a_nE[X_n]$
- $X≤Y$ with probability one, almost surely $P(X≤Y)=1, E[X]≤E[Y]]$
	![[Pasted image 20260801235756.png|285]]
- $P(a≤X≤b)=1$, then Expectation $a ≤ E[X]≤ b$ 

- $E[g(X)h(Y)]=E[g(X)]E[h(Y)]$ 
- $E[\frac{X}{Y}] ≠ \frac{E[X]}{E[Y]}$ 
	- note: $E[h(X)]≠h(E[X])$ in general, e.g. $E[\frac{1}{Y}≠\frac{1}{E[Y]}$![[07_Expectation_wHWN.jpeg|321]]



**Covariance and Correlation**
: $X, Y$ are two random variables with finite mean $\mu_X, \mu_Y$ and variances $\sigma_X^2, \sigma_Y^2$  can be calculate from marginal distribution
**Covariance**
: The average value of $\sigma_X$ from $\mu_X$ and $\sigma_Y$ from $\mu_Y$ product, can measure their association
**but**: Covariance **depends on unit / scales of $X, Y$** , ex.Height:  $m\rightarrow cm$ , $Cov * 10$ 
$g(X, Y) = (x-\mu_X)(y-\mu_Y)$ 
$$Cov(X, Y) =\sigma_{XY}= E[g(X, Y)] = E[(X-\mu_X)(Y-\mu_Y)]=E[XY]-\mu_X\mu_Y$$
- Special case: $Cov(X,X)=E[X^2]-\mu_X^2=Var(X)$ 
![[Pasted image 20260806152522.png|636]]
- Independent $\rightarrow Cov=0$ but $Cov = 0$  not $\rightarrow$ independent[[Uncorrected not independent]]
- Example: $(X_1, \cdots , X_n) \sim \text{Multinomial}(n,m,p_1, \cdots, p_m)$ then $Cov(X_i, X_j) = -np_ip_j$ [[Multinomial Covariate Example]]
why $X_i, X_j$ correlation is negative?
	when $X_i$ increasing, $X_j$ decreasing
- $Cov(a_0+\sum_{i=1}^na_iX_i, b_0+\sum_{j=1}^mb_mY_m)=\sum_{i=1}^{n}\sum_{j=1}^{m}a_ib_jCov(X_i, Y_j)$
	$=a^T \Sigma_{XY} b = (a_1, \cdots, a_n)\begin{pmatrix}\sigma_{x_1y_1} & \sigma_{x_1y_2} & \cdots & \sigma_{x_1y_m}\\ \sigma_{x_2y_1} & \sigma_{x_2y_2} & \cdots & \sigma_{x_2y_m}\\ \vdots & & \ddots & \vdots \\ \sigma_{x_ny_1} & \sigma_{x_ny_2} & \cdots & \sigma_{x_ny_m}\end{pmatrix}\begin{pmatrix}b_1\\ b_2 \\ \vdots \ b_m\end{pmatrix}$ 

- $Var(a_0+a_1X_1+\cdots+a_nX_n)=\sum_{i=1}^na^2Var(X_i)+2\sum_{1≤i≤j≤n}a_ia_jCov(X_i, X_j)$, need Joint distribution
	- if $X_i, X_j$ uncorrelated ($cov(X_i, X_j)=0, \forall i, j$)
		then $Var(a_0+a_1X_1+\cdots+a_nX_n)=\sum_{i=1}^na^2Var(X_i)$  

**Correlation Coefficient** 
: is **unit free** , measure linear relationship between $X, Y$ 
$$Cor(X, Y) =\rho_{XY}= \frac{Cov(X, Y)}{\sigma_X\sigma_Y}=E[(\frac{X-\mu_X}{\sigma_X})(\frac{Y-\mu_Y}{\sigma_Y})]$$
- $\rho_{XY}=0 \iff \sigma_{XY}=0$ 
- $Cov(X,X)=Var(X)$ 
	$=E[(X-\mu_X)(X-\mu_X)]=E[(X-\mu_X)^2]$ 
can used Correlation plot to imagine the $\text{joint PDF}$(suppose each point $f_X = \frac{1}{n}$ )![[07_Expectation_wHWN 2.jpeg]]

note:
	- Sample $X_1, X_2, \cdots, X_n$ are uncorrelated, $Var(X_1)=\cdots = Var(X_n)=\sigma=a ^2 < \infty$  $\bar X_n=\frac{X_1+\cdots+X_n}{n}$ , then $Var(\bar X_n)=\frac{\sigma^2}{n}$ 
		when $n \rightarrow \infty, \bar X_n\approx c$ (constant)  , randomize will vanishment. 
		$E[\bar X_n]=\mu$
	- Sample $X_1, X_2, \cdots, X_n$ are uncorrelated, and have same $\mu, \sigma^2$ 
		$E[S^2]=\sigma^2, S^2=\frac{\sum_{i=1}^n(X_i-\bar X_n)^2}{n-1}$ 


