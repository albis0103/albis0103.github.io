let Functional $y(x) \in [a,b]$, different between deviation is self-variable is function, to find $y(x)$ max - value or min - value
- $\mathbb{R}\rightarrow f(x) \rightarrow \mathbb{R}$ 
- $f(x)\rightarrow J[f(x)] \rightarrow \mathbb{R}$
$$J[y]=\int_a^bF(x;y, y')dx$$
### Euler - Lagrange function
$$\frac{\partial F}{\partial y}- \frac{d}{dx}\frac{\partial F'}{\partial y'}=0$$
![[IMG_3106.jpg]]
- perturbation function(noise) $\eta(x)$ : Satisfy $\eta(a)=\eta(b)=0$
- $\bar y(x)=y(x)+\epsilon \eta(x)$ 
$I(\epsilon)=\int_a^bF(x;y+\epsilon \eta, y'+\epsilon \eta')dx, I'(0)=0$
$I'(\epsilon)=\int_a^b \left[\frac{\partial F}{\partial \bar y}\eta+\frac{\partial F}{\partial \bar y'}\eta'\right]dx$ 
- $\int_a^b \frac{\partial F}{\partial \bar y'}\eta' dx=[\frac{\partial F}{\partial \bar y}\eta]_a^b-\eta \int_a^b\frac{d}{dx}\frac{\partial F}{\partial \bar y}$  
	- $u=\frac{\partial F}{\partial \bar y}, dv = \eta'$ 
	- $du=\frac{d}{dx}\frac{\partial F}{\partial \bar y}, v = \eta$ 
$I'(\epsilon)=\int_a^b \left[\frac{\partial F}{\partial \bar y}-\frac{d}{dx}\frac{\partial F}{\partial \bar y}\right]\eta(x)dx$  
$$\frac{\partial F}{\partial y}- \frac{d}{dx}\frac{\partial F'}{\partial y'}=0$$
