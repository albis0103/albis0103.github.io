透過 Generator $G$ 與 Discriminator $D$ 的零和賽局，生成出逼近資料分佈 $p_{data}$的分佈$p_g$ 
![](https://miro.medium.com/v2/resize:fit:1400/1*blPT7dNoyyV5pTvyOltBXA.png)
- Generator $G(z; \theta_g)$: 把雜訊 $z \in \mathbb{R}^{d_z}$  map到資料空間（$\mathbb{R}^{d_z}\rightarrow \mathcal{X}$）生成fake image, output $\sim p_g$ 
- Discriminator $D(x; \theta_d)$ : output $x \sim p_{data}$ ($y=1$)的機率
	- $y = 1$: sample $x$ form $p_{data}$ 
	- $y=0$: sample $x$ from $p_g$ 
### MiniMax

$$\min_{G}\max_{D}V(D,G)=\mathbb{E}_{x \sim p_{data}}[\text{log}D(x)]+\mathbb{E}_{z \sim p_{z}}[\text{log}(1-D(G(x)))]$$

- $D$: $\mathcal{X} \rightarrow \{0,1\}$ ,$D(x)$用來建模$P(y=1|x)$是二元分類問題，
- 所以單筆 Bernoulli log-likelihood：[[MLE (Maximum Likelihood Estimation)]]
$$log\;P(y|x)=y\;log\;D(x)+(1-y)\;log\;(1-D(x))$$
$$\mathbb{E}[log\;P(y|x)]=\frac{1}{2}\mathbb{E}_{x\sim p_{data}}[log\;D(x)]+\frac{1}{2}\mathbb{E}_{x\sim p_{z}}[log\;(1-D(x))]$$
so
$$V(D,G)=\int_x\left\{\;p_{data}(x)\;log\;D(x)+p_z(x)\;log\;(1-D(x))\;\right\}dx$$
Optimal Discriminator $D^*$ : $argmax_DV(D,G)$ ，$x$的值只跟$D(x)$有關，可以被$D(x)$ 獨立決定
$$\frac{\partial}{\partial D(x)}p_{data}(x)\;log\;D(x)+p_g(x)\;log\;(1-D(x))=0$$
$$\frac{p_{data}(x)}{D(x)}=\frac{p_g(x)}{1-D(x)}$$

$$D^*_G (x) = \frac{p_{data}(x)}{p_{data}(x)+p_g(x)}$$

**Training Algo**
1. $\text{init } G, \; D$
2. $\text{Fixed } G, \; \text{Updata }D$
	- 抽 $m$ 個noise $z^{(i)} \sim p_z$,  抽 $m$ 個data $x^{(i)} \sim p_{data}$
$$

\theta_d \leftarrow \theta_d + \eta,\nabla_{\theta_d}\frac{1}{m}\sum_{i=1}^{m}\Big[\log D(x^{(i)})+\log\big(1-D(G(z^{(i)}))\big)\Big]

$$
3. $\text{Fixed } D, \; \text{Updata }G$
	- 重抽 $m$ 個noise $z^{(i)} \sim p_z$
$$

\theta_g \leftarrow \theta_g - \eta,\nabla_{\theta_g}\frac{1}{m}\sum_{i=1}^{m}\log\big(1-D(G(z^{(i)}))\big)

$$
4. $\text{repeat }2.\;3.$ 

![](https://miro.medium.com/v2/resize:fit:1400/1*Jld30_Ol7hSmQ--JR2MH6A.png)

https://chsiang426.github.io/ML-2021-notes/06-Generative%20Adversarial%20Network（GAN）/06-Generative%20Adversarial%20Network（GAN）.html