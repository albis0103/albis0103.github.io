$$
\min_{x \in \mathbb{R}^n} \ \tfrac{1}{2} x^\top P x + c^\top x \quad \text{s.t.} \quad Gx \le d,\ \ h(x) = Ax-b
$$


- $P \in \mathbb{S}^n = P^T = \nabla^2f_0$ 對稱矩陣
- $P \succeq 0$（半正定，凸性條件）局部最優即全域最優。
- $P$ 為不定矩陣時是非凸 QP，一般情況下為 NP-hard。
- $h(x)$ 仿設函數



KKT 條件：
$$\tfrac12 x^\top Px + c^\top x + \lambda^\top(Gx-d) + \nu^\top(Ax-b)$$
$$
\begin{aligned}
&Px + c + G^\top \lambda + A^\top \nu = 0 && \text{（stationarity）}\\
&Gx \le h,\quad Ax = b && \text{（primal feasibility）}\\
&\lambda \ge 0 && \text{（dual feasibility）}\\
&\lambda_i (Gx - d)_i = 0,\ \ i=1,\dots,m && \text{（complementary slackness）}
\end{aligned}

$$