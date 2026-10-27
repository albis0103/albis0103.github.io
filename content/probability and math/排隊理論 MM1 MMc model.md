# M/M/1 佇列模型

**符號意義**：第一個 M 表示到達過程為 Markovian（卜瓦松過程，即到達間隔時間為指數分布），第二個 M 表示服務時間也是指數分布（Markovian），1 表示只有一台服務台。

## 基本假設

- 到達過程為卜瓦松過程，到達率為 λ（每單位時間平均到達次數）
- 服務時間服從指數分布，服務率為 μ（每單位時間平均可服務完成的顧客數）
- 單一服務台，先到先服務（FCFS）
- 系統容量無限，母體無限
- 穩定條件：ρ = λ/μ < 1（ρ 稱為利用率 utilization）

## 狀態轉移與穩態機率

系統可建模為生死過程（birth-death process），狀態 n 表示系統中有 n 個顧客。

- 出生率（到達）：λ_n = λ，對所有 n
- 死亡率（離開）：μ_n = μ，對所有 n ≥ 1

由平衡方程式（balance equations）可解得穩態機率：

$$P_n = (1-\rho)\rho^n, \quad n = 0, 1, 2, \dots$$

其中 P_0 = 1 - ρ 為系統空閒的機率。

## 關鍵績效指標

**系統內平均顧客數（含正在服務者）：**  
$$L = \frac{\rho}{1-\rho} = \frac{\lambda}{\mu - \lambda}$$

**佇列中平均等待顧客數（不含正在服務者）：**  
$$L_q = \frac{\rho^2}{1-\rho} = \frac{\lambda^2}{\mu(\mu-\lambda)}$$

**系統內平均停留時間（Little's Law 應用）：**  
$$W = \frac{L}{\lambda} = \frac{1}{\mu - \lambda}$$

**佇列中平均等待時間：**  
$$W_q = \frac{L_q}{\lambda} = \frac{\rho}{\mu-\lambda} = W - \frac{1}{\mu}$$

Little's Law：L = λW 與 L_q = λW_q 對任意穩定排隊系統皆成立，不限於 M/M/1。

---

# M/M/c 佇列模型

c 表示有 c 台平行服務台，每台服務率皆為 μ，到達仍為卜瓦松過程、服務時間仍為指數分布。

## 穩定條件

$$\rho = \frac{\lambda}{c\mu} < 1$$

此時 ρ 為每台服務台的平均利用率（非個別服務台的到達率與服務率之比）。

## 狀態轉移率

生死過程的死亡率隨狀態數而變，因為服務台數量有限：

$$\mu_n = \begin{cases} n\mu, & 0 \le n \le c \ c\mu, & n > c \end{cases}$$

（原因：當 n < c 時，僅 n 台服務台在忙碌，總服務率為 nμ；當 n ≥ c 時，全部 c 台皆忙碌，總服務率封頂在 cμ，多出的顧客需排隊等待。）

## 穩態機率

令 a = λ/μ（offered load，非 ρ）：

$$P_0 = \left[ \sum_{n=0}^{c-1} \frac{a^n}{n!} + \frac{a^c}{c!} \cdot \frac{1}{1-\rho} \right]^{-1}$$

$$P_n = \begin{cases} \dfrac{a^n}{n!} P_0, & 0 \le n < c \[2mm] \dfrac{a^n}{c! , c^{,n-c}} P_0, & n \ge c \end{cases}$$

## Erlang C 公式（顧客需等待的機率）

$$C(c, a) = P(\text{wait} > 0) = \frac{\dfrac{a^c}{c!}\cdot\dfrac{1}{1-\rho}}{\displaystyle\sum_{n=0}^{c-1}\frac{a^n}{n!} + \dfrac{a^c}{c!}\cdot\dfrac{1}{1-\rho}}$$

## 關鍵績效指標

**佇列中平均等待顧客數：**  
$$L_q = \frac{C(c,a)\cdot \rho}{1-\rho}$$

**佇列中平均等待時間：**  
$$W_q = \frac{L_q}{\lambda} = \frac{C(c,a)}{c\mu - \lambda}$$

**系統內平均停留時間：**  
$$W = W_q + \frac{1}{\mu}$$

**系統內平均顧客數：**  
$$L = \lambda W = L_q + a$$

---

## M/M/1 與 M/M/c 的架構差異對照

|項目|M/M/1|M/M/c|
|---|---|---|
|服務台數|1|c（平行）|
|死亡率結構|常數 μ|分段：nμ（n<c）與 cμ（n≥c）|
|穩定條件|λ/μ < 1|λ/(cμ) < 1|
|等待機率|1 - P_0 = ρ|Erlang C 公式 C(c,a)|
|適用場景|單通道系統（如單一 CPU 佇列、單一收銀台）|多通道系統（如多執行緒服務池、多客服窗口、多核心處理）|

M/M/c 的核心複雜度在於死亡率 μ_n 隨狀態非線性變化（因服務台數有限），這導致穩態機率的封閉解需要分段定義，且引入 Erlang C 公式來刻畫排隊機率，而非 M/M/1 中單純的幾何分布。