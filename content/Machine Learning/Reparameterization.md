
Transform the problem form 'Deviation from Random Variable' $\rightarrow$ 'Deviation from Deterministic Function'.
Ex. VAE, SGVI

### Problem
Target loss function: 
$$\mathcal{L}(\phi) = \mathbb{E}_{z \sim q_\phi(z|x)}[f(z)]$$


$$\nabla_\phi \mathcal{L}(\phi) = \nabla_\phi \int q_\phi(z|x) f(z), dz$$
Not Differentiable of $\frac{\partial z}{\partial \phi}$ because stochasticity of random variable $z = h_{\phi}(x)$.
Can not back propagation gradient  $\phi$ 
### reparameterization

Transform  $z$ to Deterministic function,  get stochasticity from $\phi$ to  $\epsilon$ :

$$z = g_\phi(\epsilon, x), \quad \epsilon \sim p(\epsilon)$$
$g_{\phi}$ is Differentiable, $p(\epsilon)$ distribution uncorrelate to $\phi$ . 



**Example**：

$q_\phi(z|x) = \mathcal{N}(z; \mu_\phi(x), \sigma_\phi^2(x))$

Reparameterization
$$z = \mu_\phi(x) + \sigma_\phi(x) \odot \epsilon, \quad \epsilon \sim \mathcal{N}(0, I)$$

Origin Loss function: $\mathcal{L}(\phi) = \mathbb{E}_{z \sim q_\phi(z|x)}[f(z)]$
Loss function可以改寫為：

$$\mathcal{L}(\phi) = \mathbb{E}_{\epsilon \sim p(\epsilon)}[f(g_\phi(\epsilon, x))]$$

因為採樣分布 $p(\epsilon)$ 不再依賴 $\phi$，梯度與期望值的順序可以互換：

$$\nabla_\phi \mathcal{L}(\phi) = \mathbb{E}_{\epsilon \sim p(\epsilon)}[\nabla_\phi f(g_\phi(\epsilon, x))]$$

因此可以用蒙地卡羅估計（單次或少量採樣 $\epsilon$）得到低變異數的無偏梯度估計，並透過標準反向傳播計算。

### 與其他梯度估計方法的比較

|方法|是否需要 $f$ 可微|變異數|適用性|
|---|---|---|---|
|重參數化（Reparameterization）|需要|低|連續型隨機變數|
|REINFORCE / Score function estimator|不需要|高|連續或離散型皆可|

REINFORCE 利用 $\nabla_\phi q_\phi(z) = q_\phi(z) \nabla_\phi \log q_\phi(z)$ 的恆等式來估計梯度，適用範圍更廣（包含離散變數），但估計器變異數通常遠高於重參數化。對於離散型隨機變數（重參數化技巧不適用），常用 Gumbel-Softmax 作為連續鬆弛近似。

### 應用場景

- **VAE**：對 $q_\phi(z|x)$ 採樣並透過 decoder $f_\theta(z)$ 重建 $x$，需要梯度同時流向 encoder 參數 $\phi$ 與 decoder 參數 $\theta$。
- **Bayes by Backprop**：對權重的後驗分布進行重參數化，估計 ELBO 梯度。
- **Normalizing Flows**：本質上是重參數化的連續組合，透過一系列可逆變換將簡單分布轉換為複雜分布。