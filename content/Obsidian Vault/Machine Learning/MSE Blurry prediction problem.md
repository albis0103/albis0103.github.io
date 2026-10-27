
In deterministic model , Regression L2-Loss will getting Blurring problem from image prediction task, because it will estimate mean value($f_{\theta}^*(x)=E[y|x]$) 

**L2-loss**$$\theta^* = \arg\min_\theta ; \mathbb{E}_{(x,y)\sim p(x,y)}\left[(y - f_\theta(x))^2\right]$$
$(x, y) \sim p(x, y)=p(x)p(y|x)$ 
$$\mathbb{E}_{(x,y)}\left[(y-f_\theta(x))^2\right] =\mathbb{E}_x\left[\mathbb{E}_{(y|x)}\left[(y-f_\theta(x))^2\right]\right]$$$$= \int p(x)\left(\int (y-f_\theta(x))^2,p(y|x)dy\right)dx$$
Fixed $x$,  Inner Layer is target function for $y$ ($\int (y-f_\theta(x))^2,p(y|x),dy$)
$$\int (y-f_\theta(x))^2p(y|x)dy=\mathbb{E}_{y|x}[(y-f(x))^2]$$
$$=E[y^2|x]-2f(x)\mathbb{E}[y|x]+f(x)^2$$
Find Extrema , Let $\frac{d}{df(x)}E[y^2|x]-2f(x)\mathbb{E}[y|x]+f(x)^2=0$ 
$$-2E(y|x)+2f(x)=0, f(x)=E(y|x)$$

![[JPEG影像-4E43-96BB-20-0.jpeg]]
- $s$: Jumping template, represent Jumping between pixel and pixel, occur at position $a \sim p(a)$ 
- $y(u)$: Observe each pixel at position $u$
- ex.$a=100, 102$
	- $u=99, s(-1)=0$
	- $u=100, s(0)=1$
	- $u=101, s(1)=2$ 
$E(y|x)=E[s(u-a)|x]=\int s(u-a)p(a|x)da=s*p$   

**Fourier Transform**
	$\mathcal{F}\{E(y|x)\}= S(\omega) \cdot \phi_{a|x}(-\omega)$
	$|\phi_{a|x}(-\omega)|=|\phi_{a|x}(\omega)|=\begin{cases}\rightarrow 0, &\text{if }\omega \text{ is larger}\\=1, & \text{if } \omega =0\end{cases}$   
	$$0 ≤ S(\omega) \cdot |\phi_{a|x}(\omega)|≤S(\omega)$$
so, $E(y|x)$ will attenuate high frequency (larger $\omega$) signal