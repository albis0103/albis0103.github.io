
核心假設
亮度恆定假設（Brightness Constancy Assumption）
：物體在影像間移動時,其**像素灰階值（在雷達上即為回波強度 dBZ）在短時間內保持不變**,只是位置改變。數學表示為：

$$I(x, y, t) = I(x+dx, y+dy, t+dt)$$

對右式做泰勒展開並取一階近似：

$$I(x+dx, y+dy, t+dt) \approx I(x,y,t) + \frac{\partial I}{\partial x}dx + \frac{\partial I}{\partial y}dy + \frac{\partial I}{\partial t}dt$$

代入恆定假設,兩邊消去 $I(x,y,t)$,兩邊除以 $dt$,得到**光流約束方程（Optical Flow Constraint Equation）**：$\frac{\partial I}{\partial x}\frac{dx}{dt} + \frac{\partial I}{\partial y}\frac{dy}{dt} + \frac{\partial I}{\partial t}\frac{dt}{dt}=0$, $(u, v)$:為$(x, y)$在$t$移動量  $dt$ ： $udt=dx, vdt=dy$


$$\frac{\partial I}{\partial x}u + \frac{\partial I}{\partial y}v + \frac{\partial I}{\partial t}=0,  \; I_x u + I_y v + I_t = 0$$

其中：

- $I_x, I_y, I_t$：影像對 x、y、t 方向的偏導數（可由差分近似）
- $u = dx/dt,\ v = dy/dt$：待求的水平、垂直移動速度分量

**問題**：這是一個方程式、兩個未知數（u, v）,無法唯一求解——這就是所謂的「**孔徑問題（Aperture Problem）**」。因此各種光流演算法的差異,本質上就是**如何加入額外約束來解這個欠定系統**。

---

### 1. Lucas-Kanade 法（稀疏光流,局部法）

**額外假設**：在一個小鄰域窗口內（例如 5×5 像素）,所有像素的 $(u,v)$ 相同。

對窗口內 n 個像素,列出 n 個光流約束方程,組成超定線性系統：

$$\begin{bmatrix} I_{x_1} & I_{y_1} \\ \vdots & \vdots \\ I_{x_n} & I_{y_n} \end{bmatrix} \begin{bmatrix} u \ v \end{bmatrix} = -\begin{bmatrix} I_{t_1} \\ \vdots \\ I_{t_n} \end{bmatrix}$$

簡寫為 $A\mathbf{v} = -b$,用最小平方法求解：

$$\mathbf{v} = (A^TA)^{-1}A^Tb$$

- $A^TA$ 需可逆,即該區域需有足夠的梯度變化（角點或邊緣附近較穩定,均勻區域會解不出來）——這也是為何稱為「稀疏」：只在特徵點附近計算。
- **優點**：計算快、對雜訊有一定抵抗力（因為是最小平方解）。
- **缺點**：只能處理小位移（因為一階泰勒展開只在小 dx, dy 下成立）；對雷達回波這種可能整體移動數公里/每時間步的情況,需搭配**金字塔（pyramid）分層**——先在低解析度（大尺度)估計大略位移,再逐層放大到原解析度修正,即 **Pyramidal Lucas-Kanade**。

---

### 2. Horn-Schunck 法（稠密光流,全域法）

**額外假設**：整個影像的運動場應該是**平滑（smooth）**的,而非局部窗口內相同。

透過最小化一個能量泛函（能量函數）來求解：

$$E(u,v) = \iint \left[ (I_xu + I_yv + I_t)^2 + \alpha^2(|\nabla u|^2 + |\nabla v|^2) \right] dx,dy$$

- 第一項：光流約束的誤差（資料項,data term）
- 第二項：平滑正則項（regularization term）,懲罰 u, v 的空間梯度過大
- $\alpha$：平滑權重,越大代表越強調全域平滑、越不信任局部資料

用變分法（Euler-Lagrange 方程）推導,最終得到迭代更新式（用拉普拉斯運算元近似鄰域平均 $\bar{u}, \bar{v}$）：

$$u^{k+1} = \bar{u}^k - \frac{I_x(I_x\bar{u}^k + I_y\bar{v}^k + I_t)}{\alpha^2 + I_x^2 + I_y^2}$$

（v 同理）

- **優點**：每個像素都能得到光流值（稠密場),適合雷達回波這種大範圍連續場,不像 L-K 只在紋理豐富處才有解。
- **缺點**：對邊界（如對流胞邊緣強度梯度大處)過度平滑會失真;計算量較大;同樣受限於小位移假設。

---

### 3. Farneback 法（稠密光流,多項式展開）

- 核心想法：用**二次多項式**局部近似每個像素鄰域的灰階分布： $$f(x) \approx x^TAx + b^Tx + c$$
- 比較前後兩張影像展開後的多項式係數,推導出局部位移估計,再用金字塔＋疊代方式處理較大位移。
- OpenCV 內建的 `calcOpticalFlowFarneback` 即此方法,在雷達 nowcasting 文獻中常被用作**傳統法 baseline**（例如與 TREC、COTREC 比較)。
- 相較 Lucas-Kanade,Farneback 天生輸出稠密場,且對中等幅度的位移處理較穩健。

---

### 4. 應用在雷達回波上的特殊處理

雷達回波場與一般自然影像有幾點差異,套用光流法時需要調整：

1. **強度非恆定**：對流胞在平移過程中同時會有增強/減弱、生成/消散,違反「亮度恆定假設」。因此常需要：

- 先對回波場做**強度歸一化**或**只取邊界/形態特徵**做光流估計,強度變化另外用外推或統計模型處理。
- 或改用**修正光流約束**,加入強度變化項 $I_xu+I_yv+I_t = \lambda$（$\lambda$ 代表允許的強度演變速率),即帶「源項（source term）」的光流模型。

2. **多尺度混合**：對流胞（小尺度,快速生成消散）與鋒面系統（大尺度,移動穩定）在同一場中並存,單一 $\alpha$ 或單一窗口大小的光流法難以同時捕捉,常見做法是**多尺度金字塔＋分區域運動場**。
    
3. **與 COTREC 的關係**：氣象領域傳統的 TREC（Tracking Radar Echoes by Correlation）本質上就是一種**區塊匹配式光流**（類似 Block Matching,而非梯度法),COTREC 則是在 TREC 之上加入類似 Horn-Schunck 的**平滑連續性約束**,可視為梯度法與區塊匹配法的融合思路。
    
4. **深度學習光流（如 PWC-Net、RAFT)**：近年 nowcasting 研究（如 Google 的 MetNet、DeepMind 的 DGMR)會用 CNN 學習運動場而非手工設計約束方程,本質上是用資料驅動方式**取代 Horn-Schunck 的正則項設計**,讓模型自己學出「什麼樣的運動場+強度演變在物理上合理」。
    

---

如果你是要對照計畫裡**起碩（洪啟碩)「雷達對流胞預測與追蹤優化」**這份報告的方法,他的追蹤演算法很可能就是用 TREC/COTREC 或類似光流概念做胞追蹤與速度場估計,我可以幫你比對報告內容跟這裡的理論對應關係。你手邊那份簡報方便的話可以貼給我看,我幫你抓對應的技術細節。