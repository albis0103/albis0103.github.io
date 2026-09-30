
傳統微積分
- $y = f(x)$, 則$y$ 微小變動：$dy = f'(x)dx$ ，因爲泰勒展開中二次項$(dx)^2$消失速度遠快於$dx$ ，所以可忽略
隨機微積分：變數(ex.利率 $r$ 或股價$S$ )包含隨機波動項 $dW$，因為$(dW)^2=dt$ ，所以泰勒展開[[Taylor Expansion]]不能省略（及伊藤引理）



**布朗運動基礎**

設 $(W_t)_{t\ge0}$ 為標準布朗運動（Wiener process），滿足：

- $W_0=0$
- 獨立增量：$W_t-W_s \perp \mathcal{F}_s$，for $t>s$
- $W_t-W_s \sim N(0,t-s)$
- 樣本路徑幾乎必然連續，但幾乎必然處處不可微，且具有無窮變異（infinite variation），二次變異 $[W]_t=t$

由於 $W_t$ 路徑不可微，無法用一般的黎曼-史蒂爾傑斯積分定義對 $dW_t$ 的積分，因此需要伊藤積分（Itô integral）。

**伊藤積分**

對一個適應過程（adapted process）$H_t$ 滿足 $E\left[\int_0^T H_s^2,ds\right]<\infty$，伊藤積分定義為

$$  
I_t=\int_0^t H_s,dW_s  
$$

透過對簡單過程的黎曼和取 $L^2$ 極限而構造，其中求和點取左端點（非中點，這是與 Stratonovich 積分的關鍵差異）。伊藤積分是一個鞅（martingale），滿足伊藤等距（Itô isometry）：

$$  
E\left[\left(\int_0^t H_s,dW_s\right)^2\right]=E\left[\int_0^t H_s^2,ds\right]  
$$

**伊藤過程的定義**

一個伊藤過程是形如

$$  
X_t = X_0+\int_0^t \mu_s,ds+\int_0^t \sigma_s,dW_s  
$$

的隨機過程，其中 $\mu_s$（漂移項，drift）與 $\sigma_s$（擴散項，diffusion）為適應過程，滿足適當的可積條件。等價地寫成微分形式（SDE）：

$$  
dX_t=\mu_t,dt+\sigma_t,dW_t  
$$

這個微分形式只是上述積分方程的簡寫，本身沒有逐點意義（因為 $dW_t$ 不存在古典導數）。

---

## 伊藤公式 (Itô's Formula)

**動機**：對於一般函數 $f$，若 $X_t$ 為伊藤過程，古典鏈式法則不適用，因為需要考慮 $dW_t$ 的二次變異項 $(dW_t)^2=dt$（在均方意義下）。

**一維伊藤公式**

設 $X_t$ 為上述伊藤過程，$f(t,x)\in C^{1,2}$（對 $t$ 一階可微，對 $x$ 二階可微），則

$$  
df(t,X_t)=\frac{\partial f}{\partial t}dt+\frac{\partial f}{\partial x}dX_t+\frac{1}{2}\frac{\partial^2 f}{\partial x^2}(dX_t)^2  
$$

代入 $dX_t=\mu_t,dt+\sigma_t,dW_t$，並使用伊藤乘法表：

$$  
dt\cdot dt=0,\qquad dt\cdot dW_t=0,\qquad dW_t\cdot dW_t=dt  
$$

得到

$$  
df(t,X_t)=\left(\frac{\partial f}{\partial t}+\mu_t\frac{\partial f}{\partial x}+\frac{1}{2}\sigma_t^2\frac{\partial^2 f}{\partial x^2}\right)dt+\sigma_t\frac{\partial f}{\partial x},dW_t  
$$

**推導核心**：對 $f(t,X_t)$ 作泰勒展開至二階：

$$  
df=f_t,dt+f_x,dX_t+\frac{1}{2}f_{xx}(dX_t)^2+\frac{1}{2}f_{tt}(dt)^2+f_{tx},dt,dX_t+\cdots  
$$

其中 $(dt)^2$ 與 $dt,dX_t$ 為高階小量（$o(dt)$），可忽略；但 $(dX_t)^2=\sigma_t^2(dW_t)^2+2\mu_t\sigma_t,dt,dW_t+\mu_t^2(dt)^2\to\sigma_t^2,dt$（因 $(dW_t)^2$ 的均方極限為 $dt$），這是伊藤公式與古典微積分的本質差異來源。

**多維伊藤公式**

設 $\mathbf{X}_t=(X_t^1,\dots,X_t^n)$ 為向量伊藤過程，$dX_t^i=\mu_t^i,dt+\sum_{j=1}^m \sigma_t^{ij},dW_t^j$，$f(t,\mathbf{x})\in C^{1,2}$，則

$$  
df(t,\mathbf{X}_t)=\frac{\partial f}{\partial t}dt+\sum_{i=1}^n\frac{\partial f}{\partial x_i}dX_t^i+\frac{1}{2}\sum_{i,k=1}^n\frac{\partial^2 f}{\partial x_i\partial x_k},d\langle X^i,X^k\rangle_t  
$$

其中二次共變異 $d\langle X^i,X^k\rangle_t=\left(\sum_j \sigma_t^{ij}\sigma_t^{kj}\right)dt$。

---

## 典型應用：幾何布朗運動 (GBM)

設 $S_t$ 滿足 $dS_t=\mu S_t,dt+\sigma S_t,dW_t$（Black-Scholes 模型的標的資產動態）。令 $f(x)=\ln x$，套用伊藤公式：

$$  
f_x=\frac{1}{x},\quad f_{xx}=-\frac{1}{x^2}  
$$

$$  
d(\ln S_t)=\frac{1}{S_t}dS_t-\frac{1}{2S_t^2}(dS_t)^2=\left(\mu-\frac{\sigma^2}{2}\right)dt+\sigma,dW_t  
$$

積分後得到閉式解：

$$  
S_t=S_0\exp\left[\left(\mu-\frac{\sigma^2}{2}\right)t+\sigma W_t\right]  
$$

這裡的 $-\sigma^2/2$ 修正項正是伊藤公式二階項的直接產物，是與古典 ODE 解法的關鍵差異。

---

需要我接著推導 Girsanov 定理（測度轉換）、Feynman-Kac 公式（PDE 與 SDE 的對應），或是伊藤積分的嚴格構造（從簡單過程到 $L^2$ 極限）嗎？