: Conception is **Step-by-step Learning Denoise-process**, let model generate image from noise

**Forwarding Process, Diffusion Process**
: Add the noise gradually to let image reach complete noise state
![[Pasted image 20260810220412.png]]
It is parameterized Markov chain, add Gaussian noise gradually.
	gevin item $x_0 \sim q(x_0)$ 
	$q(x_t \mid x_{t-1}) := \mathcal{N}(x_t; \sqrt{1-\beta_t} x_{t-1},\ \beta_t I)$
		$\beta_t \in (0, 1)$:Hyperparameter,  $i_{th}$ noise schedule , control noise insertion
		$t=1, \cdots, T$
Through Reparameterization and Gaussian Distribution Additivity, can calculate $t$ marginal distribution [[Reparameterization]]

Let 
	$\alpha_t = 1-\beta_t$, $\sqrt{\alpha}$  is because  $Var(cX)=c^2Var(X)$ 
Reperameterization: $x_t = \sqrt{\alpha_t} x_{t-1} + \sqrt{1-\alpha_t}\cdot \epsilon_1, \qquad \epsilon \sim \mathcal{N}(0, I)$
	then can calculate $x_t$ without $x_{t-1}\cdots x_1$ $$x_t = \sqrt{\bar\alpha_t} x_0 + \sqrt{1-\bar\alpha_t}\cdot \epsilon, \;\bar\alpha_t = \prod_{s=1}^t \alpha_s$$  
	- because $x_{t} = \sqrt{\alpha_{t}\alpha_{t-1}\cdots \alpha_{0}} x_{0} + \sqrt{1-\alpha_{t}\alpha_{t-1}\cdots \alpha_{0}}\cdot \epsilon$
		- $x_t = \sqrt{\alpha_t} x_{t-1} + \sqrt{1-\alpha_t}\cdot \epsilon_1, \qquad \epsilon \sim \mathcal{N}(0, I)$ 
			- $x_{t-1} = \sqrt{\alpha_{t-1}} x_{t-1} + \sqrt{1-\alpha_{t-1}}\cdot \epsilon_{2}$ ...
		$\epsilon_1 \sim \mathcal{N}(0, 1-\alpha_t), \epsilon_2 \sim \mathcal{N}(0, \alpha_t(1-\alpha_{t-1})$
		$\mathcal{N}(0, \sigma_1^2 I)+\mathcal{N}(0, \sigma_2^2I) = \mathcal{N}(0, (\sigma_1^2+\sigma_2^2)I)$ 
		
	
$$q(x_t \mid x_0) = \mathcal{N}(x_t;\ \sqrt{\bar\alpha_t}, x_0,\ (1-\bar\alpha_t) I)$$
$$q(x_{1:T}|x_0)=\prod_{t=1}^Tq(x_t|x_{t-1})$$



when $T \to \infty$ $\bar\alpha_T \to 0$， $x_T \approx \mathcal{N}(0, I)$。


**Reverse Process**
: Reverse the process, model learning transformation between noise and image
![[Pasted image 20260810220540.png]]
used $p_{\theta}$ estimate $q$ 
![492](https://miro.medium.com/v2/resize:fit:700/1*gVNa41KLwu3GZ5kF4wRUOg.png)
	by **Bayesian + Markov**
	Posterior
	$q(x_{t-1} \mid x_t, x_0) = \mathcal{N}(x_{t-1};\ \tilde\mu_t(x_t, x_0),\ \tilde\beta_t I)$
		$$= \frac{q(x_t \mid x_{t-1}, x_0)q(x_{t-1}\mid x_0)}{q(x_t \mid x_0)}$$
		-  $q(x_t \mid x_{t-1})=\sqrt{\alpha_{t-1}}x_t+\sqrt{(1-\alpha_{t-1})\epsilon}$   
			$\sim \mathcal{N}(\sqrt{\alpha_{t-1}} x_0, (1-\alpha_{t-1}))\propto \exp\left(-\frac{(x_t - \sqrt{\alpha_t},x_{t-1})^2}{2\beta_t}\right)$ 
		- $q(x_{t-1} \mid x_0) \propto \exp\left(-\frac{(x_{t-1} - \sqrt{\bar\alpha_{t-1}},x_0)^2}{2(1-\bar\alpha_{t-1})}\right)$
		- $q(x_t \mid x_0) \propto \exp\left(-\frac{(x_t - \sqrt{\bar\alpha_t},x_0)^2}{2(1-\bar\alpha_t)}\right)$
	$$=\exp\left(-\frac{1}{2}\left(-\frac{(x_t - \sqrt{\alpha_t},x_{t-1})^2}{\beta_t}+\right)\right)$$
		$$=\exp\left(-\frac{1}{2}\left[\left(\frac{\alpha_t}{\beta_t} + \frac{1}{1-\bar\alpha_{t-1}}\right)x_{t-1}^2 - 2\left(\frac{\sqrt{\alpha_t}}{\beta_t}x_t + \frac{\sqrt{\bar\alpha_{t-1}}}{1-\bar\alpha_{t-1}}x_0\right)x_{t-1} + C(x_t, x_0)\right]\right)$$
		$\tilde\mu_t(x_t, x_0) = \frac{\sqrt{\alpha_t}(1-\bar\alpha_{t-1})}{1-\bar\alpha_t}x_t + \frac{\sqrt{\bar\alpha_{t-1}},\beta_t}{1-\bar\alpha_t}x_0$
		$\tilde\beta_t = \frac{1-\bar\alpha_{t-1}}{1-\bar\alpha_t}\beta_t$ 
	
	$p(x_{t-1} \mid x_t) \propto q(x_t \mid x_{t-1})p(x_{t-1})$
	$:= \mathcal{N}\big(x_{t-1};\ \mu_\theta(x_t, t),\ \Sigma_\theta(x_t, t)\big)$
		$t = T \dots 1$ 
			start from $x_T \sim \mathcal{N}(0,I)$ sequential : $x_{T-1}, x_{T-2}, \dots, x_0$ 
	by $q(x_t \mid x_0) = \mathcal{N}(x_t;\ \sqrt{\bar\alpha_t}, x_0,\ (1-\bar\alpha_t) I)$
	$q(x_{t-1} \mid x_0) = \mathcal{N}(x_{t-1};\ \sqrt{\bar\alpha_t}, x_0,\ (1-\bar\alpha_{t-1}) I)$ 
	
**Loss Function**	$$\mathcal{L} = \mathbb{E}_q\Big[D_{\mathrm{KL}}\big(q(x_{t-1}\mid x_t, x_0)||p_\theta(x_{t-1}\mid x_t)\big)\Big] + c$$
		by $x_t = \sqrt{\bar\alpha_t} x_0 + \sqrt{1-\bar\alpha_t}\cdot \epsilon, \qquad \epsilon \sim \mathcal{N}(0, I), \epsilon_{\theta}(x_t, t)$ 
	$$L_{\text{simple}}(\theta) = \mathbb{E}_{t,, x_0,, \epsilon}\Big[\big|\epsilon - \epsilon_\theta(x_t, t)\big|^2\Big]$$ 