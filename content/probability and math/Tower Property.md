

 $(\Omega, \mathcal{F}, P)$ 為機率空間，$\mathcal{G} \subseteq \mathcal{H} \subseteq \mathcal{F}$ 為子 σ-代數。則：
 $$E\big[E[X \mid \mathcal{H}] \big| \mathcal{G}\big] =E\big[E[X \mid \mathcal{G}] \big| \mathcal{H}\big] $$
正向
$$  
E\big[E[X \mid \mathcal{H}] \big| \mathcal{G}\big] = E[X \mid \mathcal{G}] \quad \text{a.s.}  
$$

反方向：

$$  
E\big[E[X \mid \mathcal{G}] \big| \mathcal{H}\big] = E[X \mid \mathcal{G}] \quad \text{a.s.}  
$$

**結果等於只對較小的那個取條件期望。**

## 2. 常用形式

以隨機變數表示資訊時（$\mathcal{G} = \sigma(X)$，$\mathcal{H} = \sigma(X, Z)$）：

$$  
E\big[,E[Y \mid X, Z] ,\big|, X\big] = E[Y \mid X]  
$$

取 $\mathcal{G} = {\emptyset, \Omega}$（trivial σ-代數，$E[\cdot \mid \mathcal{G}] = E[\cdot]$）時，得到最常用的特例：

$$  
E\big[E[Y \mid X]\big] = E[Y]  
$$

## 3. 證明

**第一式。** 令 $Z = E[X \mid \mathcal{H}]$。要證 $E[Z \mid \mathcal{G}] = E[X \mid \mathcal{G}]$

- $E[X \mid \mathcal{G}]$ 是 $\mathcal{G}$-可測（由定義）。
- 對任意 $A \in \mathcal{G}$，因 $\mathcal{G} \subseteq \mathcal{H}$，有 $A \in \mathcal{H}$，所以

$$  
\int_A Z dP = \int_A X  dP = \int_A E[X \mid \mathcal{G}]  dP  
$$

前面是 $Z$ 作為 $E[X \mid \mathcal{H}]$ 的定義，後面用的是 $E[X \mid \mathcal{G}]$ 的定義。由條件期望的 a.s. 唯一性得證。

**第二式。** $E[X \mid \mathcal{G}]$ 是 $\mathcal{G}$-可測，因此也是 $\mathcal{H}$-可測。對一個 $\mathcal{H}$-可測的變數取 $E[\cdot \mid \mathcal{H}]$ 會得到它本身。

**離散特例的直接驗證：**

$$  
E\big[E[Y \mid X]\big] = \sum_x P(X=x) \sum_y y , P(Y=y \mid X=x) = \sum_x \sum_y y , P(X=x, Y=y) = E[Y]  
$$

## 4. 幾何解釋（$L^2$ 投影）

若 $X \in L^2$，$E[X \mid \mathcal{G}]$ 是 $X$ 在閉子空間 $L^2(\mathcal{G})$ 上的正交投影。由 $\mathcal{G} \subseteq \mathcal{H}$ 可得 $L^2(\mathcal{G}) \subseteq L^2(\mathcal{H})$。記投影算子為 $\Pi_{\mathcal{G}}$ 與 $\Pi_{\mathcal{H}}$，則

$$  
\Pi_{\mathcal{G}} \Pi_{\mathcal{H}} = \Pi_{\mathcal{H}} \Pi_{\mathcal{G}} = \Pi_{\mathcal{G}}  
$$

也就是說，先投影到大子空間再投影到其內的小子空間，等同直接投影到小子空間。

## 5. 應用

**(a) 全變異數公式（Law of Total Variance）**

$$  
\operatorname{Var}(Y) = E\big[\operatorname{Var}(Y \mid X)\big] + \operatorname{Var}\big(E[Y \mid X]\big)  
$$

推導：

- $E[\operatorname{Var}(Y|X)] = E\big[E[Y^2|X]\big] - E\big[(E[Y|X])^2\big] = E[Y^2] - E\big[(E[Y|X])^2\big]$，第二個等號用了 tower property。
- $\operatorname{Var}(E[Y|X]) = E\big[(E[Y|X])^2\big] - (E[Y])^2$，其中 $E\big[E[Y|X]\big] = E[Y]$ 同樣來自 tower property。

兩式相加即得。

**(b) 迴歸中 $E[y \mid x]$ 的最優性**

對任意 $f_\theta$：

$$  
E_{x,y}\big[(y - f_\theta(x))^2\big] = E_x\Big[E\big[(y - f_\theta(x))^2 ,\big|, x\big]\Big]  
$$

這裡用 tower property 把外層期望拆成兩層。內層在給定 $x$ 下對常數 $f_\theta(x)$ 取最小，最小值出現在 $f_\theta(x) = E[y \mid x]$。展開後的交叉項為 $E_x\big[(E[y|x] - f_\theta(x)) \cdot E[y - E[y|x] \mid x]\big] = 0$，這一步也依賴 tower property。

**(c) Martingale**

若 $\mathcal{F}_n$ 是 filtration，$M_n = E[X \mid \mathcal{F}_n]$，則對 $m \le n$：

$$  
E[M_n \mid \mathcal{F}_m] = E\big[E[X \mid \mathcal{F}_n] ,\big|, \mathcal{F}_m\big] = E[X \mid \mathcal{F}_m] = M_m  
$$

因此 ${M_n}$ 是 martingale（Doob martingale），這一點直接由 tower property 得出。

## 6. 注意事項

- **巢狀條件不可省略。** 若 $\mathcal{G} \not\subseteq \mathcal{H}$，一般而言 $E\big[E[X|\mathcal{H}] \mid \mathcal{G}\big] \neq E[X|\mathcal{G}]$。
- **需要 $E|X| < \infty$**，條件期望才有定義。
- 等式是 **almost surely** 成立，不是 pointwise。