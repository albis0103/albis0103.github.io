The integral of $g$ over $(a, b]$ is defined:
$$\int_a^bg(x)dx = lim\sum_{i=1}^ng(x_i)(x_i-x_{i-1})$$
Unit of Interval:  $(x_i-x_{i-1})\rightarrow [F(x_i)-F(x_{i-1})]$ 
Riemann-Stieltjes Integral
$$\int_a^b f(x), dg(x)=lim\sum_{i=1}^ng(x_i)[F(x_i)-F(x_{i-1})]$$
where the limit is taken over $a = x_0 < x_1 < \cdots < x_n = b$ as $n \rightarrow \infty$ and $max_{i=1\cdots n}(x_i-x_{i-1}\rightarrow 0)$


## 存在性的關鍵條件

**基本定理**：若 $f$ 在 $[a,b]$ 上連續，且 $g$ 在 $[a,b]$ 上為有界變差（bounded variation），則 $\int_a^b f, dg$ 存在。

更精確的充分條件（互補性）：$f$ 與 $g$ 不能在同一點同時不連續。若 $g$ 在點 $c$ 有跳躍不連續，只要 $f$ 在 $c$ 連續，積分仍存在。

## 與 Lebesgue–Stieltjes 積分的關係

若 $g$ 是遞增函數，$g$ 可誘導出一個測度 $\mu_g$（對半開區間 $(a,b]$ 定義 $\mu_g((a,b]) = g(b)-g(a)$），則

$$\int_a^b f, dg = \int_{[a,b]} f , d\mu_g$$

此為 Lebesgue–Stieltjes 積分，適用範圍更廣（$f$ 不必連續，只需可測且可積）。

## 分部積分公式

$$\int_a^b f, dg + \int_a^b g, df = f(b)g(b) - f(a)g(a)$$

這是 R-S 積分相對 Riemann 積分的一個重要優勢：能自然處理兩函數角色互換的情形。

## 若 $g$ 可微

若 $g \in C^1[a,b]$，則

$$\int_a^b f(x), dg(x) = \int_a^b f(x) g'(x), dx$$

化簡為一般 Riemann 積分乘以密度函數 $g'$。

## 主要應用

- **機率論／統計**：若 $g = F$ 為隨機變數的累積分佈函數（CDF），則 $$E[f(X)] = \int_{-\infty}^{\infty} f(x), dF(x)$$ 此式同時涵蓋離散（$F$ 為階梯函數）與連續（$F$ 可微，$dF = f_X(x)dx$）分佈，是統一處理兩者的關鍵工具。
    
- **測度論的橋樑**：R-S 積分是從古典分析過渡到測度論／Lebesgue 積分的自然銜接點，尤其在建構機率測度時常見。
    
- **訊號處理／控制理論**：處理含 Dirac delta（脈衝）的系統響應時，可將其視為積分子 $g$ 的跳躍點。
    

## 一個具體例子

設 $f(x) = x^2$，$g(x)$ 為階梯函數： $$g(x) = \begin{cases} 0, & x < 1 \ 3, & 1 \le x < 2 \ 5, & x \ge 2 \end{cases}$$

在 $[0,3]$ 上，$g$ 僅在 $x=1$ 與 $x=2$ 有跳躍（跳躍量分別為 3 與 2）。由於 $f$ 連續，

$$\int_0^3 f, dg = f(1)\cdot 3 + f(2)\cdot 2 = 1\cdot 3 + 4\cdot 2 = 11$$

（一般規則：若 $g$ 只有可數個跳躍點 $c_k$，跳躍量 $j_k$，且連續部分導數為 $g'_c$，則 $\int f,dg = \sum_k f(c_k)j_k + \int f(x)g'_c(x),dx$。）

如果你是在準備研究所的分析課或機率論基礎，我可以針對某個方向（例如與測度論的嚴謹連接，或分部積分定理的證明）再展開。