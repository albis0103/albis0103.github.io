
$$\mathcal{F}{f(t)} = F(\omega) = \int_{-\infty}^{\infty} f(t)e^{-i\omega t}dt$$

$$\mathcal{F}^{-1}{F(\omega)} = f(t) = \frac{1}{2\pi}\int_{-\infty}^{\infty} F(\omega)e^{i\omega t}d\omega$$


- condition：$f(t)$ integrable $\int_{-\infty}^\infty |f(t)|dt < \infty$
 - $t \in (-\infty,\infty)$

## Common Transform

| $f(t)$                       | $F(\omega)$                                              |
| ---------------------------- | -------------------------------------------------------- |
| $\delta(t)$                  | $1$                                                      |
| $1$                          | $2\pi\delta(\omega)$                                     |
| $\cos(\omega_0 t)$           | $\pi[\delta(\omega-\omega_0)+\delta(\omega+\omega_0)]$   |
| $\sin(\omega_0 t)$           | $i\pi[\delta(\omega+\omega_0)-\delta(\omega-\omega_0)]$  |
| $e^{-a\lvert t\rvert},\ a>0$ | $\dfrac{2a}{a^2+\omega^2}$                               |
| $e^{-at}u(t),\ a>0$          | $\dfrac{1}{a+i\omega}$                                   |
| $u(t)$                       | $\pi\delta(\omega)+\dfrac{1}{i\omega}$                   |

## Key Property

- Linear: $\mathcal{F}{af(t)+bg(t)} = aF(\omega)+bG(\omega)$
- Time shift: $\mathcal{F}{f(t-t_0)} = e^{-i\omega t_0}F(\omega)$
- Frequency shift (modulation): $\mathcal{F}{e^{i\omega_0 t}f(t)} = F(\omega-\omega_0)$
- Time scaling: $\mathcal{F}{f(at)} = \dfrac{1}{|a|}F!\left(\dfrac{\omega}{a}\right)$
- Differentiation: $\mathcal{F}{f^{(n)}(t)} = (i\omega)^n F(\omega)$
- Integration: $\mathcal{F}\{\int_{-\infty}^t f(\gamma)d\gamma\}= \frac{F(\omega)}{i\omega} + \pi F(0)\delta(\omega)$  
- Convolution: $\mathcal{F}{f*g} = F(\omega)G(\omega)$
- Duality: $\mathcal{F}{F(t)} = 2\pi f(-\omega)$
- Parseval's theorem: $\displaystyle\int_{-\infty}^{\infty}|f(t)|^2dt = \frac{1}{2\pi}\int_{-\infty}^{\infty}|F(\omega)|^2d\omega$

## 求解流程（與 Laplace 對應）

Laplace 解 ODE 是走「代數化 → 部分分式（residue）→ 查表反轉」。Fourier 在你的用途上（訊號/頻譜分析）通常不解初值問題，而是：

1. 對已知訊號 $f(t)$ 直接查表或用性質組合求 $F(\omega)$
2. 分析頻域特性（頻寬、濾波器響應、能量分布 via Parseval）
3. 若需反轉，同樣可用留數法對 $F(\omega)$ 做圍線積分（Bromwich-type contour），但常用情況多半直接查表

### Example：頻域求解一階微分方程

$$y'(t) + ay(t) = x(t), \quad a>0$$

1. 兩邊取 Fourier（假設穩態，無初值項，這是與 Laplace 最大差異處——Fourier 沒有 $f(0)$ 這種初值修正項）：

$$i\omega Y(\omega) + aY(\omega) = X(\omega)$$

2. 整理：

$$Y(\omega) = \frac{X(\omega)}{a+i\omega} = H(\omega)X(\omega)$$

其中 $H(\omega) = \dfrac{1}{a+i\omega}$ 稱為系統的**頻率響應（transfer function 在虛軸上的取值）**

3. 若 $x(t)=\delta(t)$，則 $X(\omega)=1$，查表得脈衝響應：

$$y(t) = h(t) = e^{-at}u(t)$$

4. 驗證：代回 $y'(t)+ay(t)$，對 $t>0$ 得 $-ae^{-at}+ae^{-at}=0$ ✓，在 $t=0$ 的跳躍對應 $\delta(t)$ ✓

## 核心差異總結

||Laplace|Fourier|
|---|---|---|
|積分域|$[0,\infty)$|$(-\infty,\infty)$|
|變數|$s=\sigma+i\omega$|$i\omega$（純虛軸）|
|處理初值|內建於微分性質中|不處理，預設穩態/全時域已知|
|主要用途|暫態分析、穩定性（極點位置）|頻譜分析、濾波器設計|
|反轉工具|留數定理 + 查表|查表為主，必要時亦可用留數|