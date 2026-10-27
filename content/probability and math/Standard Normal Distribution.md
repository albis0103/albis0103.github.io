


$$Z = \frac{X - \mu}{\sigma} \sim N(0, 1)$$

## PDF

$$\varphi(z) = \frac{1}{\sqrt{2\pi}} , e^{-z^2/2}, \quad z \in (-\infty, \infty)$$

性質：

- 正規化：$\int_{-\infty}^{\infty} \varphi(z),dz = 1$
- 偶函數：$\varphi(-z) = \varphi(z)$
- 導數：$\varphi'(z) = -z,\varphi(z)$
- $E[Z] = 0$，$\mathrm{Var}(Z) = 1$，偏態 0，峰度 3

注意：PDF 的值是密度而非機率，單點機率 P(Z = z) = 0；機率由區間積分得到。

## 2. CDF

$$\Phi(z) = P(Z \le z) = \int_{-\infty}^{z} \varphi(t),dt = \int_{-\infty}^{z} \frac{1}{\sqrt{2\pi}} e^{-t^2/2},dt$$

此積分**沒有初等函數的封閉形式**，需用數值計算或查表。可用誤差函數表示：

$$\Phi(z) = \frac{1}{2}\left[1 + \mathrm{erf}!\left(\frac{z}{\sqrt{2}}\right)\right]$$

其中 $\mathrm{erf}(x) = \frac{2}{\sqrt{\pi}}\int_0^x e^{-t^2},dt$。

性質：

- $\Phi'(z) = \varphi(z)$
- $\Phi(0) = 0.5$
- 對稱性：$\Phi(-z) = 1 - \Phi(z)$
- 單調遞增，$\Phi(-\infty) = 0$，$\Phi(\infty) = 1$
- 區間機率：$P(a < Z \le b) = \Phi(b) - \Phi(a)$
- 反函數 $\Phi^{-1}(p)$ 稱為分位函數（probit），用於求分位數

## 3. 常用數值

|z|Φ(z)|
|---|---|
|0|0.5000|
|1|0.8413|
|1.645|0.9500|
|1.96|0.9750|
|2|0.9772|
|3|0.9987|

經驗法則（68-95-99.7）：

- $P(|Z| \le 1) \approx 0.6827$
- $P(|Z| \le 2) \approx 0.9545$
- $P(|Z| \le 3) \approx 0.9973$
