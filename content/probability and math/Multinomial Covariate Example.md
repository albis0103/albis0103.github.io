
$X = (X_1, \cdots , X_m) \sim \text{Multinomial}(n,m,p_1, \cdots, p_m)$

$(X_1,X_2 ,X_3+\cdots+ X_n) \sim \text{Multinomial}(n,3,p_1, p_2, p_3\cdots+p_n)$
	$X_3+\cdots+X_n = n-X_1-X_2$
	$p_3+\cdots +p_n=1-p_1p_2$
so
$f_X = \binom{n}{x_1, x_2, n-x_1-x_2}p_1^{x_1}p_2^{x_2}(p_3+\cdots +p_n)^{1-p_1p_2}$
$$E[X_1X_2]=\sum_{0≤x_1+x_2≤n, x_1≠0, x_2≠0} x_1x_2\binom{n}{x_1, x_2, n-x_1-x_2}p_1^{x_1}p_2^{x_2}(p_3+\cdots +p_n)^{(n-x_1x_2)}$$
	$$=\sum_{2≤x_1+x_2≤n} x_1x_2\frac{n!}{x_1!x_2!n-x_1-x_2!}p_1^{x_1}p_2^{x_2}(p_3+\cdots +p_n)^{(n-x_1x_2)}$$
	$$= n(n-1)\sum_{x_1-1+x_2-1≤n-2}\frac{(n-2)!}{(x_1-1)!(!n-x_1-x_2)!}p_1^{x_1}p_2^{x_2}(p_3+\cdots +p_n)^{(n-x_1x_2)}$$
	$=n(n-1)p_1p_2$
$Cov(X_i, X_j)=E[X_i X_j]-\mu_{X_i}\mu_{X_j}$
	$=n(n-1)p_ip_j-(np_i)(np_j)=-np_ip_j$
	