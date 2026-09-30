
$h:\mathbb{R}^m \rightarrow \mathbb{R}^1$
$$E(h(Y)|X=x)=\sum_{y\in \mathcal{Y}}h(y)p_{Y|X}(y|x)$$
$$E(h(Y)|X=x)=\int_{y\in \mathcal{Y}}h(y)f_{Y|X}(y|x)$$

$x^* = \text{Fixed } x$
$f(x^*, y)$ is NOT PDF, $\int f(x^*, y)dy ≠ 1$, $\frac{f(x^*, y)}{f_X(x^*)}=1$ is PDF $f_{Y|X}(y|x^*)$ 
$E(Y|x^*)$ = mean of $f_{Y|X}(y|x^*)$
generalize $x* \rightarrow \forall x$ , $E(Y|x)$ is function
![[07_Expectation_wHWN 3.jpeg|291]]


by Tower Property
- $E_X\{E_{Y|X}[h(x)|X]\}=E_Y[h(Y)]$
- $E_X\{E_{Y|X}[Y_i|X]\}=E_Y[Y_i]$ ![[07_Expectation_wHWN 4.jpeg]]
$E_X\{E_{Y|X}[h(Y)|X]\}$
$$=\int_{\mathbb{R}^n}E_{Y|X}[h(Y)|X]f_X(x)dx$$
$$=\int_{\mathbb{R}^n}[\int_{\mathbb{R}^m}h(y)f_{Y|X}(y|x)dy]f_X(x)dx$$
$$=\int_{\mathbb{R}^m}\int_{\mathbb{R}^n} h(y)\frac{f_{XY}(x,y)}{f_X(x)}f_X(x)dxdy$$
$$=\int_{\mathbb{R}^m}\int_{\mathbb{R}^n} h(y){f_{XY}(x,y)}dxdy=E_{XY}(h(y))$$ 
$$=\int_{\mathbb{R}^m}h(y)f_Y(y)dy=E_Y(h(y))$$