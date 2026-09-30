

|      | $dt$             | $dX$                               |
| ---- | ---------------- | ---------------------------------- |
| $dt$ | $dt^2=0$         | $dt \cdot dX = dt^{\frac{3}{2}}=0$ |
| $dX$ | $dx\cdot dt = 0$ | $dt$                               |

### 1. 設定：分割與增量

取區間 $[0,t]$ 的分割 $\Pi: 0=t_0<t_1<\cdots<t_n=t$，令  
$$\Delta t_i = t_{i+1}-t_i, \qquad \Delta W_i = W_{t_{i+1}}-W_{t_i} \sim \mathcal{N}(0,\Delta t_i)$$  
且各 $\Delta W_i$ 彼此獨立（由布朗運動的獨立增量性質）。網目 $|\Pi| = \max_i \Delta t_i$。

---

### 2. 為什麼 $(dW)^2 = dt$：二次變差收斂證明

定義  
$$I_n = \sum_{i=0}^{n-1} (\Delta W_i)^2$$

**計算期望值：**  
$$E[I_n] = \sum_i E[(\Delta W_i)^2] = \sum_i \Delta t_i E[Z_i^2]$$
$$\Delta t_i = E[(\Delta W_i)^2]$$

**計算變異數：** 
$$Var(I_n) = \sum_iVar((\Delta W)i)^2) = \sum_i \Delta t_iVar(Z_i^2)$$
已知常態分布平方變異數$Var(Z_i^2) = 2$（因為四階動差$E[Z^4]=3, Var(Z^2) = E[Z^4]-(E[Z^2])^2=3-1=2$）


$$Var(I_n) = 2\sum_i \Delta t_i \;≤\;2max(\Delta t_i)\sum_i\Delta t_i = 2max(\Delta t_i)\cdot T$$

最大區隔長度$max(\Delta t_i)\rightarrow 0$則
$$lim_{n \rightarrow \infty }Var(I_n) \rightarrow t$$

所以$(dW)^2=dt$ 

---

### 3. 為什麼 $dt \cdot dW = 0$

定義交叉項  
$$C_\Pi = \sum_i \Delta t_i, \Delta W_i$$

**期望值：** 由 $E[\Delta W_i]=0$，  
$$E[C_\Pi] = \sum_i \Delta t_i, E[\Delta W_i] = 0$$

**變異數：** 由獨立性，  
$$\text{Var}(C_\Pi) = \sum_i (\Delta t_i)^2,\text{Var}(\Delta W_i) = \sum_i (\Delta t_i)^3 \le |\Pi|^2 \sum_i \Delta t_i = |\Pi|^2, t \xrightarrow{|\Pi|\to 0} 0$$

故 $C_\Pi \xrightarrow{L^2} 0$，即混合變差恆為零。

---

### 4. 為什麼 $(dt)^2 = 0$

此項為確定性（非隨機）量，不需機率論工具：  
$$\sum_i (\Delta t_i)^2 \le |\Pi| \sum_i \Delta t_i = |\Pi|, t \xrightarrow{|\Pi|\to 0} 0$$

這是一般 Riemann 積分中「二階微分可忽略」的標準事實，布朗運動的隨機性在此不起作用。

---

### 5. 統一觀點：階數分析（heuristic order counting）

由 $\text{Var}(\Delta W_i) = \Delta t_i$，可將 $\Delta W_i$ 視為 $O(\sqrt{\Delta t_i})$ 階量（因其標準差之尺度為 $\sqrt{\Delta t_i}$）。據此：

|乘積|階數|對 $\sum_i(\cdot)$ 極限行為|
|---|---|---|
|$\Delta t_i \cdot \Delta t_i$|$O(\Delta t_i^2)$|$\to 0$（比 $\Delta t_i$ 高一階）|
|$\Delta t_i \cdot \Delta W_i$|$O(\Delta t_i^{3/2})$|$\to 0$（比 $\Delta t_i$ 高半階）|
|$\Delta W_i \cdot \Delta W_i$|$O(\Delta t_i)$|**不消失**，收斂至 $\Delta t_i$ 本身|

只有 $(\Delta W_i)^2$ 與 $\Delta t_i$ 同階，因此在取極限（$|\Pi|\to 0$，同時分割數 $n\to\infty$）時，唯有 $(dW)^2$ 這一項會貢獻與 $dt$ 相同量級的極限值；其餘兩項的階數嚴格高於一階，故求和後消失。

---

### 6. 此結果的應用：Itô 公式的來源

對 $f\in C^2$，Taylor 展開 $f(W_{t+\Delta t}) - f(W_t)$ 至二階：  
$$\Delta f = f'(W_t)\Delta W + \tfrac{1}{2}f''(W_t)(\Delta W)^2 + \tfrac{1}{2}f''(W_t)\cdot o(\Delta t) + \cdots$$

代入 $(\Delta W)^2 \to \Delta t$（依上述 $L^2$ 收斂），求和取極限後即得  
$$df(W_t) = f'(W_t),dW_t + \tfrac{1}{2}f''(W_t),dt$$

這正是 Itô 公式中額外出現 $\tfrac{1}{2}f'',dt$ 修正項的來源——它源自於一般微積分中被忽略的二階項 $(\Delta W)^2$，在布朗運動的情形下並不趨於零，而是趨於 $dt$。