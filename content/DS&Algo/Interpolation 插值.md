: Given Discrete items $(x_0, y_0), (x_1, y_1), \cdots, (x_n, y_n)$ , find the function 
	$f(x), \forall i \; f(x_i)=y_i$ 
to estimate Unknown value  between item

- **Polynomial Interpolation**
	:N個點找到N個多項式係數

	note: unisolvence theorem
		Ｎ個點存在唯一 N-1 次多項式通過所有點(Weierstrass/ Vandermonde 唯一性)
![[Pasted image 20260730111255.png|532]]
		
	
- Vandermonde Matrix
	- 給 n 個函數點 ($x_1, x_2, \cdots, x_n$), Vandermonde Matrix $V_{ij}=x_i^{j-1}$:
	- 基底：$\{1, x, \cdots, x^{n-1}\}$
	- $$V = \begin{pmatrix} 1 & x_0^1 & x_0^2 & \cdots & x_0^{n-1} \\ 1 & x_1^1 & x_1^2 & \cdots & x_1^{n-1} \\ \vdots & \vdots & \vdots & \ddots & \vdots \\ 1 & x_{n-1}^1 & x_{n-1}^2 & \cdots & x_{n-1}^{n-1} \end{pmatrix}, \det(V)=\prod_{1\leq i<j\leq n}(x_j-x_i)$$ 
	- interpolation function:
			$f(x_0)=c_0+c_1x_0^1+\cdots+c_{n-1}x_0^{n-1}$ 
			$f(x_0)=c_0+c_1x_1^1+\cdots+c_{n-1}x_1^{n-1}$ 
			...
			條件：$Vc=y, c=V^{-1}y$ 唯一存在
	$\begin{bmatrix}c_0 \\ c_1 \\ c_2 \\ c_3 \\ c_4\end{bmatrix}=\begin{pmatrix}  1 & x_0^1 & x_0^2 & \cdots & x_0^{n-1} \\  1 & x_1^1 & x_1^2 & \cdots & x_1^{n-1} \\ \vdots & \vdots & \vdots & \ddots & \vdots \\  1 & x_{n-1}^1 & x_{n-1}^2 & \cdots & x_{n-1}^{n-1}  \end{pmatrix}^{-1}\begin{bmatrix}f(x_0) \\ f(x_1)\\ f(x_2) \\ \vdots \\ f(x_{n-1})\end{bmatrix}$
	
			

 - Lagrange interpolation : $n$個 $n-1$ 次多項式
		$1^{st}$只穿過$1^{st}點$，$2^{nd}$只穿過$2^{nd}點, \cdots,$ ，$n^{th}$只穿過$n^{th}點$
		then 相加($\sum$)且穿過所有點（唯一性：n個相異點決定 ≤n-1 次多項式）


	基底:$\{l_1, l_2, \cdots, l_n\}, l_i(x_j)=\prod_{j=1 \&j≠i}^n\frac{x-x_j}{x_i-x_j}$ 
		
 	![[Pasted image 20260727154137.png]]
		$$L(x)\sum_{i=0}^{n-1}y_i\cdot l_i(x_j)=\sum_{i=0}^{n-1}y_i\prod_{j=1 \&j≠i}^n\frac{x-x_j}{x_i-x_j}$$
	- Newton interpolation: 透過差商(Divided Differences)遞推，逐次加入新的點，更新內差函數
		![[Pasted image 20260727161644.png]]
		1. 穿過 $n$個點的內差函數
		2. 前$n$ 點$=0$ ，只穿過$n+1^{th}$點
		3. 兩函數相加
		4. recursive
		差商 Divided Differences
			given $(x_0, y_0), \cdots, (x_n, y_n)$ 
			- $0$階：$f[x_i]=y_i$ 
			- $1$階：$f[x_i, x_{i+1}]=\frac{f[x_{i=1}]-f[x_i]}{x_{i+1}-x_i}$ 
			- $k$階(recursive)：$f[x_i, \cdots, x_{i+k}]=\frac{f[x_i, \cdots, x_{i+k}]-f[x_i, \cdots, x_{i+k-1}]}{x_{i+k}-x_i}$ 
		Newton Interpolation:
		$p(x) = f[x_0]+f[x_0, x_1]+\cdots =\sum_{k=0}^nf[x_0, \cdots, x_k]\prod_{j=0}^{k-1}(x-x_j)$
	
- **Piecewise Interpolation**: Interpolation fumction 是多個 $0$次多項式
	- linear interpolation
	- quadratic interpolation
	- monotone cubic interpolation
	- spline interpolation
- **Basis Interpolation**
	- RBF interpolation(Radial Basis Function)
		- $\Phi(x) = \exp\left(\frac{|x-x_0|^2}{2\sigma^2}\right), \quad \sigma \text{ is constant}$$$\begin{bmatrix}\Phi(\lVert x_0 - x_0 \rVert) & \Phi(\lVert x_0 - x_1 \rVert) & \cdots & \Phi(\lVert x_0 - x_{N-1} \rVert) \\ \Phi(\lVert x_1 - x_0 \rVert) & \Phi(\lVert x_1 - x_1 \rVert) & \cdots & \Phi(\lVert x_1 - x_{N-1} \rVert) \\ \vdots & \vdots & \ddots & \vdots \\ \Phi(\lVert x_{N-1} - x_0 \rVert) & \Phi(\lVert x_{N-1} - x_1 \rVert) & \cdots & \Phi(\lVert x_{N-1} - x_{N-1} \rVert) \end{bmatrix} \begin{bmatrix} w_0 \\ w_1 \\ \vdots \\ w_{N-1} \end{bmatrix} = \begin{bmatrix} y_0 \\ y_1 \\ \vdots \\ y_{N-1} \end{bmatrix}$$
	- ![[Pasted image 20260825101950.png|313]]
