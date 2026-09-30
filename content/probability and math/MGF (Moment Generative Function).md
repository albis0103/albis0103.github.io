: Describe the all **moments** of random variable $X$, is  let $s = -t,M_X(t)= \mathcal{L}\{f_X(x)\}(-t)$ 
[[Laplace Transform]]
$$M_X(t)=\mathcal{L}\{f_X(x)\}=E[e^{tX}]  = \int_{-\infty}^{\infty}e^{sX}f_X(x)dx$$ Another Describe $f(x)$ used Taylor Expansion View an let $\text{Number of }a_k \rightarrow \text{UnCountable}$ 
- In Traditional, $y = f(x)$
- In Taylor, $f(x)=\sum_{k=0}^{\infty} a_k x^k,\;a_k=\frac{f^{(k)}(0)}{k!}, \; k=0, 1, .., \in \mathbb{N}$ countable
- In Laplace, $k \in \mathbb{R}$ Uncountable
![[07_Expectation_wHWN 6.jpeg]]
![[07_Expectation_wHWN 7.jpeg]]note: $M_X(t)$ is Taylor expansion $t$
	$M_X(t)=E[e^{tX}]=E\left[\sum_{k=0}^{\infty}\frac{(tX)^k}{k!}\right]=\sum_{k=0}^{\infty}\frac{E[X^k]}{k!},t^k$ 

Key property: Suppose $M_X(t)$ and $M_Y(t)$ for r.v $X,Y$ exist for all $|t|<h$ for some $h>0$ 
- **Uniqueness**: $M_X(t)=M_Y(t)$ then $F_X(z)=F_Y(z)$ one to one mapping
	- When $MGF$ existed, there had unique distribution correspond. Uniqueness commonly used for Linear Combination of Independent r.v 
		- $M_X(t)=p_1e^{a_1t}+\cdots p_ke^{a_kt}, p_i=p_X(x), \sum p_i = 1$ , then 
			- ans: discrete r.v. $X$ pmf is $p_X(x)=\begin{cases}p_i, &\text{for }x=a_i, \; i=1,..,k\\0, &\text{otherwise}\end{cases}$ 
- **Linear Transformation:**
	- $M_{aX+b}(t) = e^{bt}M_X(at)$
	- $S=X_1 + \cdots + X_n, X_1\cdots X_n$ are independent r.v. with mgf $M_1(t),\cdots, M_n(t)$ 
		- $M_S(t) = M_1(t)\times \cdots \times M_n(t)$ 
- **Moments and MGF:** $M_X(0)=1, M_X^{(n)}(0)=\mu_n=E[X^n]$ [[Moment]]
	- because $M_X(t)=\sum_{n=0}^{\infty}\frac{t^n}{n!}E[X^n]$, then let  $K_X(t)= ln M_X(t)$  
	- $K'(0)=E[X]$, because $K'(0)=\frac{M_X'(0)}{M_X(0)}=\frac{E[X]}{1}$
	- $K''(0)=Var(X)$ because $K''(0)= \frac{\partial M_X'(0)/M_X(0)}{\partial t}=\frac{M_X''(0)M_X(0)-M_X'(0)^2}{M_X'(t)^2}=\frac{E[X^2]-E[X]^2}{1}$
- Let r.v $X_1, \cdots X_n$ independent, and $S = \sum_{i=1}^nX_i$ 
	then $M_S(t)=\prod_{t=1}^nM_X(t)$ 
- $M_X(it)=\phi_X(t)$
- 