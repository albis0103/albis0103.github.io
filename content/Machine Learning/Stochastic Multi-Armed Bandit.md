https://hackmd.io/@shaoeChen/Syp8clcd_/https%3A%2F%2Fhackmd.io%2F%40shaoeChen%2FSk7frzq4u
## 1. 問題設定

共 $k$ 個 action。在每個 time step $t = 1, 2, \dots$：

1. agent 選擇 action $A_t$
2. 得到 reward $R_t$

定義 action $a$ 的真實 value：

$$  
q_*(a) \doteq \mathbb{E}[R_t \mid A_t = a]  
$$

「Stochastic」指給定 $A_t = a$ 時，$R_t$ 服從固定分佈，其期望值 $q_*(a)$ 不隨 $t$ 改變。agent 只能觀察到被選 action 的 reward，不知道 $q_*(a)$。

目標是最大化期望累積 reward $\mathbb{E}\big[\sum_{t} R_t\big]$。若 $q_*(a)$ 已知，最佳策略就是恆選 $\arg\max_a q_*(a)$。

## 2. Value 估測

令 $Q_t(a)$ 為 action $a$ 在 time step $t$ 的估測 value，目標是 $Q_t(a) \to q_*(a)$。sample-average 方法：

$$  
Q_t(a) \doteq \frac{\sum_{i=1}^{t-1} R_i \cdot \mathbb{1}_{A_i = a}}{\sum_{i=1}^{t-1} \mathbb{1}_{A_i = a}}  
$$

其中 $\mathbb{1}_{A_i = a}$ 在 $A_i = a$ 時為 1，否則為 0。分母即 $N_t(a)$，為 action $a$ 在 $t$ 之前被選擇的次數。由大數法則，$N_t(a) \to \infty$ 時 $Q_t(a) \to q_*(a)$。

增量形式（$Q_n$ 為該 action 被選 $n-1$ 次後的估測）：

$$  
Q_{n+1} = Q_n + \frac{1}{n}\big[R_n - Q_n\big]  
$$

形式為 $\text{NewEstimate} \leftarrow \text{OldEstimate} + \text{StepSize},[\text{Target} - \text{OldEstimate}]$。

## 3. Exploration vs. Exploitation

- **Greedy action**：$A_t \doteq \arg\max_a Q_t(a)$。選 greedy action 是 exploitation，選其他 action 是 exploration。
- 純 greedy 會因早期估測偏差而可能永遠不再選到真正最佳的 action。

## 4. 方法

### 4.1 $\varepsilon$-greedy

$$  
A_t \doteq \begin{cases} \arg\max_a Q_t(a) & \text{機率 } 1-\varepsilon \\ \text{均勻隨機選 } a & \text{機率 } \varepsilon \end{cases}  
$$

每個 action 都會被無限次選到，因此 $Q_t(a) \to q_*(a)$。

### 4.2 Optimistic Initial Values

設 $Q_1(a)$ 高於實際可能的 $q_*(a)$，使 greedy 在早期必須嘗試每個 action。這只提供早期的一次性探索。

### 4.3 UCB

$$  
A_t \doteq \arg\max_a \left[ Q_t(a) + c\sqrt{\frac{\ln t}{N_t(a)}} \right]  
$$

- $c > 0$ 控制探索程度
- $N_t(a) = 0$ 時，視 $a$ 為 maximizing action
- 根號項衡量 $Q_t(a)$ 的不確定性：$N_t(a)$ 越小，項越大

### 4.4 Gradient Bandit

不估 $q_*(a)$，改學 preference $H_t(a)$，policy 為 softmax：

$$  
\pi_t(a) \doteq \Pr{A_t = a} = \frac{e^{H_t(a)}}{\sum_{b=1}^{k} e^{H_t(b)}}  
$$

令 $\bar{R}_t$ 為截至 $t$ 的平均 reward（baseline），更新為：

$$  
H_{t+1}(A_t) \doteq H_t(A_t) + \alpha\big(R_t - \bar{R}_t\big)\big(1 - \pi_t(A_t)\big)  
$$

$$  
H_{t+1}(a) \doteq H_t(a) - \alpha\big(R_t - \bar{R}_t\big)\pi_t(a), \quad \forall a \neq A_t  
$$

這是 $\mathbb{E}[R_t]$ 對 $H_t(a)$ 做 gradient ascent 的 stochastic 近似。

## 5. Nonstationary 情形

若 $q_*(a)$ 隨 $t$ 變動，改用固定 step-size $\alpha \in (0, 1]$：

$$  
Q_{n+1} = Q_n + \alpha\big[R_n - Q_n\big] = (1-\alpha)^n Q_1 + \sum_{i=1}^{n} \alpha(1-\alpha)^{n-i} R_i  
$$

近期 reward 的權重呈指數衰減（exponential recency-weighted average）。