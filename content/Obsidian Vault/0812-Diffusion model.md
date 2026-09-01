# Diffusion Model

## 核心概念

Diffusion model 是一種生成模型，透過兩個相反的隨機過程學習資料分布：

1. **前向過程（forward process）**：逐步對資料加入高斯雜訊，直到訊號完全被破壞成標準常態分布
2. **反向過程（reverse process）**：學習一個參數化模型，逐步去除雜訊，從純雜訊還原出資料樣本

## 前向過程

給定資料 $x_0 \sim q(x_0)$，前向過程定義為一個馬可夫鏈，逐步加入高斯雜訊：

$$q(x_t \mid x_{t-1}) = \mathcal{N}(x_t; \sqrt{1-\beta_t}, x_{t-1},\ \beta_t I)$$

其中 $\beta_t \in (0,1)$ 是第 $t$ 步的雜訊排程（noise schedule），$t = 1, \dots, T$。

利用重參數化與高斯分布的可加性，可以直接推導出任意時刻 $t$ 的邊際分布（closed form），不需逐步採樣：

令 $\alpha_t = 1-\beta_t$，$\bar\alpha_t = \prod_{s=1}^t \alpha_s$，則

$$q(x_t \mid x_0) = \mathcal{N}(x_t;\ \sqrt{\bar\alpha_t}, x_0,\ (1-\bar\alpha_t) I)$$

等價地，可寫成重參數化形式：

$$x_t = \sqrt{\bar\alpha_t}, x_0 + \sqrt{1-\bar\alpha_t},\epsilon, \qquad \epsilon \sim \mathcal{N}(0, I)$$

當 $T \to \infty$ 且排程設計合理時，$\bar\alpha_T \to 0$，故 $x_T \approx \mathcal{N}(0, I)$。

## 反向過程

真實的反向條件分布 $q(x_{t-1} \mid x_t)$ 不可解析求得（依賴整個資料分布），因此用神經網路 $p_\theta$ 來近似：

$p(x_{t-1} \mid x_t) \propto q(x_t \mid x_{t-1}), p(x_{t-1})$
	$:= \mathcal{N}\big(x_{t-1};\ \mu_\theta(x_t, t),\ \Sigma_\theta(x_t, t)\big)$

生成過程即從 $x_T \sim \mathcal{N}(0,I)$ 開始，依序採樣 $x_{T-1}, x_{T-2}, \dots, x_0$。

## 訓練目標

利用貝氏定理，可以推導出後驗分布 $q(x_{t-1} \mid x_t, x_0)$ 同樣是高斯分布，其均值與變異數皆有解析解：

$$q(x_{t-1} \mid x_t, x_0) = \mathcal{N}(x_{t-1};\ \tilde\mu_t(x_t, x_0),\ \tilde\beta_t I)$$

其中

$$\tilde\mu_t(x_t, x_0) = \frac{\sqrt{\alpha_t}(1-\bar\alpha_{t-1})}{1-\bar\alpha_t}x_t + \frac{\sqrt{\bar\alpha_{t-1}},\beta_t}{1-\bar\alpha_t}x_0$$

$$\tilde\beta_t = \frac{1-\bar\alpha_{t-1}}{1-\bar\alpha_t}\beta_t$$

原始目標是最大化資料的對數似然，透過變分下界（ELBO）分解為每個時間步的 KL 散度：

$$\mathcal{L} = \mathbb{E}_q\Big[D_{\mathrm{KL}}\big(q(x_{t-1}\mid x_t, x_0),|,p_\theta(x_{t-1}\mid x_t)\big)\Big] + \text{常數項}$$

由於兩者皆為高斯分布，KL 散度可化簡為均值間的加權平方誤差。將重參數化 $x_t = \sqrt{\bar\alpha_t}x_0 + \sqrt{1-\bar\alpha_t}\epsilon$ 代入，可以把預測均值的問題轉換成**預測雜訊** $\epsilon$ 的問題，即用網路 $\epsilon_\theta(x_t, t)$ 估計混入 $x_t$ 中的雜訊。

Ho et al. (2020, DDPM) 發現忽略 KL 中的權重係數、直接用簡化的均方誤差損失，訓練效果更好且更穩定：

$$L_{\text{simple}}(\theta) = \mathbb{E}_{t,, x_0,, \epsilon}\Big[\big|\epsilon - \epsilon_\theta(x_t, t)\big|^2\Big]$$

## 採樣

訓練完成後，反向採樣使用以下更新式（以 DDPM 為例）：

$$x_{t-1} = \frac{1}{\sqrt{\alpha_t}}\left(x_t - \frac{\beta_t}{\sqrt{1-\bar\alpha_t}}\epsilon_\theta(x_t, t)\right) + \sigma_t z, \qquad z \sim \mathcal{N}(0, I)$$

其中 $\sigma_t^2$ 通常取 $\beta_t$ 或 $\tilde\beta_t$。

## 常見變體

- **DDIM**：將反向過程改寫為非馬可夫的確定性採樣，可大幅減少採樣步數
- **Score-based generative model (SDE 觀點)**：Song et al. 證明 diffusion 的離散化過程對應到隨機微分方程，$\epsilon_\theta$ 的預測等價於估計 score function $\nabla_x \log q(x_t)$
- **Classifier-free guidance**：透過同時訓練條件與無條件模型，在採樣時外插以增強生成品質與可控性

需要我針對某個部分（例如 ELBO 完整推導、score-based 觀點、或 DDIM 的推導）展開更詳細的說明嗎？