
# Vasicek 模型

Vasicek 模型是描述短期利率動態的單因子均值回歸隨機模型，由 Oldřich Vašíček 於 1977 年提出，屬於 Ornstein-Uhlenbeck 過程的應用。

## 隨機微分方程

短期利率 $r_t$ 的動態由下式描述：

$$dr_t = a(b - r_t),dt + \sigma,dW_t$$

其中：

- $a > 0$：均值回歸速度（mean reversion speed）
- $b$：長期均衡利率水準（long-run mean）
- $\sigma > 0$：波動率參數
- $W_t$：標準布朗運動

漂移項 $a(b - r_t)$ 使得 $r_t > b$ 時利率有下降傾向，$r_t < b$ 時有上升傾向，形成均值回歸特性。

$$dW = \epsilon\cdot \sqrt{dt}$$

## 解析解

此 SDE 為線性 SDE，可解得：

$$r_t = r_0 e^{-at} + b(1 - e^{-at}) + \sigma \int_0^t e^{-a(t-s)},dW_s$$

由於是高斯過程之線性組合，$r_t$ 服從常態分配：

$$r_t \mid r_0 \sim \mathcal{N}\left(r_0 e^{-at} + b(1-e^{-at}),\ \frac{\sigma^2}{2a}\left(1 - e^{-2at}\right)\right)$$

當 $t \to \infty$ 時，穩態分配為：

$$r_\infty \sim \mathcal{N}\left(b,\ \frac{\sigma^2}{2a}\right)$$

## 零息債券定價

在此模型下，零息債券價格具封閉解：

$$P(t,T) = A(t,T),e^{-B(t,T),r_t}$$

其中：

$$B(t,T) = \frac{1 - e^{-a(T-t)}}{a}$$

$$A(t,T) = \exp\left[\left(b - \frac{\sigma^2}{2a^2}\right)\big(B(t,T) - (T-t)\big) - \frac{\sigma^2}{4a}B(t,T)^2\right]$$

此為仿射期限結構模型（affine term structure model）的典型代表。

## 主要限制

1. **利率可為負值**：因 $r_t$ 服從常態分配，支撐集為 $(-\infty, \infty)$，這在低利率環境出現前曾被視為主要缺陷。
2. **單因子**：只有一個布朗運動驅動，隱含所有到期日利率完全相關，無法產生殖利率曲線的複雜形態變化。
3. **常數參數**：$a, b, \sigma$ 皆設為常數，缺乏對殖利率曲線初始形狀的完全配適能力（相對於 Hull-White 模型的時變 $b(t)$ 擴展）。

## 與相關模型的比較

|模型|SDE|利率可為負|解析解|
|---|---|---|---|
|Vasicek|$dr_t = a(b-r_t)dt + \sigma dW_t$|可|有|
|CIR|$dr_t = a(b-r_t)dt + \sigma\sqrt{r_t},dW_t$|否（Feller 條件下）|有（非中心卡方分配）|
|Hull-White|$dr_t = a(\theta(t)-r_t)dt + \sigma dW_t$|可|有|

CIR 模型透過將擴散項改為 $\sigma\sqrt{r_t}$ 解決負利率問題，但犧牲了條件分配的常態性；Hull-White 模型則將 $b$ 擴展為時變函數 $\theta(t)$ 以完全配適初始殖利率曲線。

需要我進一步推導債券價格公式的完整過程，或說明參數估計方法（如最大概似估計）嗎？