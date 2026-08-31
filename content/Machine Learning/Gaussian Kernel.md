
**Machine Learning(SVM)**

$k(x_i, x_j) = e^{-\gamma||x_i-x_j||^2}$
$=e^{-\frac{||x_i-x_j||^2}{2\sigma^2}} = exp(-\frac{||x_i||^2-2x_i^Tx_j+||x_j||^2}{2\sigma^2})$   

	$e^{\frac{x_i^Tx_j}{\sigma^2}}=\sum_{n=0}^{\infty}\frac{(x_i^Tx_j)^n}{\sigma^{2n}n!}=\sum_{n=0}^{\infty} \frac{\langle x_i^{\otimes n}, x_j^{\otimes n} \rangle}{\sigma^{2n} n!}$
		$=\sum_{n=0}^{\infty} \frac{1}{\sigma^{2n} n!}\langle x_i^{\otimes n}, x_j^{\otimes n} \rangle =\sum_{n=0}^{\infty} \frac{1}{\sigma^{n} \sqrt{n!}}\frac{1}{\sigma^{n} \sqrt{n!}}\langle x_i^{\otimes n}, x_j^{\otimes n} \rangle$  
	
$k(x_i, x_j) = e^{-\frac{||x_i||^2+||x_j||^2}{2\sigma^2}}\sum_{n=0}^{\infty} \frac{1}{\sigma^{n} \sqrt{n!}}\frac{1}{\sigma^{n} \sqrt{n!}}\langle x_i^{\otimes n}, x_j^{\otimes n} \rangle$
	$= e^{-\frac{||x||^2}{2\sigma^2}}\sum_{n=0}^{\infty}\langle \frac{ x_i^{\otimes n}}{\sigma^{n} \sqrt{n!}}\frac{x_j^{\otimes n}}{\sigma^{n} \sqrt{n!}}  \rangle$
		$\text{by polynomial kernel} \langle x, y\rangle^n= \sum_{|\alpha| = n}x^{\alpha}y^{\alpha}=\langle \phi_n(x), \phi_n(y)\rangle$
	$=e^{-\frac{||x||^2}{2\sigma^2}}[\frac{x^{\otimes 0}}{\sigma^0\sqrt{0!}},\frac{x^{\otimes 1}}{\sigma^1\sqrt{1!}},\frac{x^{\otimes 2}}{\sigma^2\sqrt{2!}},..,\frac{x^{\otimes \infty}}{\sigma^{\infty}\sqrt{\infty!}}]=\phi(x)$ 


**Gaussian Blurring**

used 2-D Gaussian Distribution to create blur kernel for blurry picture
1. Calculate kernel size $(6\sigma+1)(6\sigma+1)$		
2. used $G(x,y)$ Calculate kernel parameter
		2-D Gaussian : $G(\mathbf{x}) = \frac{1}{2\pi|\Sigma|^{1/2}}\exp\left(-\frac{1}{2}(\mathbf{x}-\mu)^T\Sigma^{-1}(\mathbf{x}-\mu)\right)$ 
		let $\Sigma = \sigma^2 I = \begin{bmatrix} \sigma^2 & 0 \\ 0 & \sigma^2 \end{bmatrix}$, $|\Sigma|=\sigma^2$ , $\Sigma^{-1}=\frac{1}{\sigma^2}$ 
		$G(x, y) = \frac{1}{2\pi\sigma^2}e^{-\frac{x^2+y^2}{2\sigma^2}}$ 
3. used kernel to blurry picture