$\text{Conditional Distribution}=\frac{Joint}{Marginal}$ 
### PMF
![[Pasted image 20260726212414.png|291]]
$$p_{Y|X}(y|x)=\frac{p_{X, Y}(x, y)}{p_X(x)}$$ 
$$\sum_y p_{Y|X}(y|x) = \sum_y \frac{p_{X, Y}(x, y)}{p_X(x)}=\frac{1}{p_X(x)}\sum_yp_{X, Y}(x, y)=\frac{1}{p_X(x)}p_X(x)=1$$
  


- for event $B$ of $Y$, $Y \in B$ given $X=x$ 
	$P(Y \in B|X = x) = \sum_{u \in B}P(Y=u|X=x)=\sum_{u\in B}p_{Y|X}(u|x)$ 
- for event $B = \{Y_1≤y_1, \cdots , Y_m≤y_m\}$ can define $\text{Conditional joint CDF}$ 
	$F_{Y|X}(y|x)=P(Y≤y|X=x) = \sum_{u≤y}p_{Y|X}(u|x)$ 

- Let $X_1,\cdots, X_n$ independent and $X_i \sim Poisson(\lambda_i)$
	$Y = X_1 + \cdots X_n$ then will transform to $Multinomial$ 
	$(X_1,\cdots, X_n|Y=n) \sim Multinomial(n,m,p_1, \cdots, p_m)$ [[Multinomial Distribution]]
	$p_i=\frac{\lambda_i}{\lambda_1+\cdots+\lambda_m}$ for $i=1,\cdots,m$
	Proof:
		$(X_1,\cdots, X_n|Y=n)\text{pmf}=\frac{p_{X,Y}(x_1,\cdots,x_m,n)}{p_Y(n)}$
			$p_{X,Y}(x_1,\cdots,x_m,n)=\begin{cases}P(X_1=x_1 ,\cdots , X_m=x_m), & if \; x_1+\cdots+x_m=n,\\ 0,  & if \; x_1+\cdots+x_m≠n, \end{cases}$
			$p_Y(n)=P(Y=n)=\frac{e^{- \lambda_1 + \cdots + \lambda_m}(\lambda_1 + \cdots + \lambda_m)^n}{n!}$  
		$p_{X|Y}(x,y)=\frac{p_{X,Y}(x_1,\cdots,x_m,n)}{p_Y(n)}=\frac{\prod_{i=1}^m \frac{e^{-\lambda_i \lambda_i^{x_i}}}{x_i!}}{\frac{e^{- \lambda_1 + \cdots + \lambda_m}(\lambda_1 + \cdots + \lambda_m)^n}{n!}}$
			$=\frac{n!}{x_1!*\cdots*x_m!}*(\frac{\lambda_1}{\lambda_1+\cdots+\lambda_m})^{x_1}*\cdots*(\frac{\lambda_m}{\lambda_1+\cdots+\lambda_m})^{x_m}$ 
### PDF
![[Pasted image 20260726212520.png|311]]
$$f_{Y|X}(y|x)=\frac{f_{Y|X}(y|x)}{f_X(x)}, X \in \mathbf{R}^n, Y\in \mathbf{R}^m$$ 
- Proof: 
	$\{x_i-\frac{\Delta x_i}{2} < x_i≤ x_i+\frac{\Delta x_i}{2}\}$ 
	$f_{Y|X}(y|x):=\frac{\partial }{\partial y}F_{Y|X}(y|x)$ 
		$F_{Y|X}(y|x)=P(Y≤y|x-\frac{\Delta x}{2}<X≤ x+\frac{\Delta x}{2})$ 
			$=\frac{\int_{-\infty}^y\int_{x-\frac{\Delta x}{2}}^{x+\frac{\Delta x}{2}}f_{X,Y}(u, v)dudv}{\int_{x-\frac{\Delta x}{2}}^{x+\frac{\Delta x}{2}}f_{X}(t)dt}\approx \frac{\int_{-\infty}^yf_{X, Y}(x, v)\Delta x dv}{f_X(x)\Delta x}=\frac{\int_{-\infty}^yf_{X, Y}(x, v)dv}{f_X(x)}$ 