Given Observe item $(x_i, y_i), i=1, \cdots, n$ , Least square method used min Residual sum of square to find the best parameter $\hat \theta$ 
$$\hat \theta = arg\; min_\theta \sum_{i=1}^n(y-f(x))^2$$
linear: $f_{\theta}(x)=\theta^Tx$
$$\hat \theta = arg\; min_\theta||y-X\theta||_2^2,\; X \in \mathbb{R}^{n \times p}, y \in \mathbb{R}^n$$
	ex. $x=1,2,3, y=2,3,5$
	$y_i = \alpha + \beta x_i \rightarrow \begin{cases}\alpha + \beta(1)=2\\\alpha + \beta(2)=3\\\alpha + \beta(4)=5\end{cases}\rightarrow  {\begin{pmatrix}2\\3\\5\\\end{pmatrix}} = {\begin{pmatrix}1&1\\1&2\\1&3\\ \end{pmatrix}}{\begin{pmatrix}\alpha\\\beta\end{pmatrix}}$ 
**Normal Equation**:$(X^TX)\theta=X^Ty$ 

$$\nabla_{\theta} ||y-\theta X||=0$$
$$\frac{\partial }{\partial \theta }(y-\theta X)(y - \theta X)^T=yy^T-2\theta Xy^T + \theta XX^T \theta^T=0$$ 
$$-2X^T(y-X\theta )=0$$
$$(X^TX)\theta=X^Ty$$
$$\hat \theta =(X^TX)^{-1}X^Ty$$
