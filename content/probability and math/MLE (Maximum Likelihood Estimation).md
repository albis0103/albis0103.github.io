$$X = \{x^t\}_{i=1}^N, \; x^t \sim p(x|\theta)\quad i.i.d$$
**sampling $x^t$  from $p(x|\theta)$ as likely as possible**
- 找到一組 $\theta$ 讓 model best-fit sample data.

MLE: 找出 $\theta^*$的過程
- 找出所有 outcomes （ $N$ instances $x^t$ ，彼此獨立）形成之distribution 的 function 時，function 的參數是$\theta$ 的機率
$$l(\theta|X) \equiv p(X|\theta)=\prod_{t=1}^Np(x^t|\theta)$$
or 
$$\mathcal{L}(\theta|X)\equiv log\;l(\theta|X)=\sum_{t-1}^Nlog\;p(x^t|\theta)$$

ex. **Bernoulli**
$$p(x)=p^x(1-p)^{1-x},\; \mu=p, \; \sigma=p(1-p)$$
$$\mathcal{L}(p|X)=log\;\prod_{t=1}^N p^{x^t}(1-p)^{(1-p^{x^t})}$$
$$=\sum_t x\cdot log\;(p)+(N-\sum_tx)log\;(1-p)$$
- note : $\frac{d}{dp}log(1-p) = -\frac{1}{1-p}$ 
$$ \frac{\partial}{\partial p}\mathcal{L} = \sum x\cdot \frac{1}{p} - (N-\sum x)\frac{1}{1-p}=0$$
$$\sum x\cdot \frac{1}{p} =(N-\sum x)\frac{1}{1-p}, \;p^*=\frac{\sum x}{N} = \mu$$
