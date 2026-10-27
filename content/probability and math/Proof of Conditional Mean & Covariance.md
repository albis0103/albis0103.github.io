
$$ \mathbf{x}_A = f(x), \qquad \mathbf{x}_B = f(x_{1:n}) $$


## 設定 
$$ \begin{bmatrix} \mathbf{x}_A \\ \mathbf{x}_B \end{bmatrix} \sim \mathcal{N}\!\left(\begin{bmatrix} \boldsymbol{\mu}_A \\ \boldsymbol{\mu}_B \end{bmatrix},\; \begin{bmatrix} \Sigma_{AA} & \Sigma_{AB} \\ \Sigma_{BA} & \Sigma_{BB} \end{bmatrix}\right), \qquad \Sigma_{BA} = \Sigma_{AB}^\top $$

| 通用符號 | GP 符號 |
|--- | --- |
| $\boldsymbol{\mu}_A$ | $\mu_0(x)$ |
| $\boldsymbol{\mu}_B$ | $\mu_0(x_{1:n})$ |
| $\Sigma_{AA}$ | $\Sigma_0(x, x)$ |
| $\Sigma_{AB}$ | $\Sigma_0(x, x_{1:n})$ |
| $\Sigma_{BA}$ | $\Sigma_0(x_{1:n}, x)$ |
| $\Sigma_{BB}$ | $\Sigma_0(x_{1:n}, x_{1:n})$ |
| $\boldsymbol{\mu}_{A\mid B}$ | $\mu_n(x)$ |
| $\Sigma_{A\mid B}$ | $\sigma_n^2(x)$ |
目標：求 $\mathbf{x}_A \mid \mathbf{x}_B$ 的分布。

**結論（兩種推導共同的結果）：**

$$ \mathbf{x}_A \mid \mathbf{x}_B \sim \mathcal{N}(\boldsymbol{\mu}_{A|B},, \Sigma_{A|B}) $$

$$ \boldsymbol{\mu}_{A|B} = \boldsymbol{\mu}_A + \Sigma_{AB}\Sigma_{BB}^{-1}(\mathbf{x}_B - \boldsymbol{\mu}_B), \qquad \Sigma_{A|B} = \Sigma_{AA} - \Sigma_{AB}\Sigma_{BB}^{-1}\Sigma_{BA} $$

---

## 推導一：構造獨立變數

$$\begin{pmatrix}\mathbf{x}_A\\\mathbf{x}_B\end{pmatrix}=\mathcal{N}\left(\begin{pmatrix}\mu_A\\ \mu_B\end{pmatrix},\begin{pmatrix}\Sigma_{AA} & \Sigma_{AB}\\\Sigma_{BA} & \Sigma_{BB}\end{pmatrix}\right)$$
**Step 1： $\mathbf{z} = \mathbf{x}_A - C\mathbf{x}_B$，$\operatorname{Cov}(\mathbf{z}, \mathbf{x}_B) = 0$，與$\mathbf{x}_B$無關($\mathbf{z} \perp \mathbf{x}_B$)

$$ \operatorname{Cov}(\mathbf{z}, \mathbf{x}_B)=Cov(\mathbf{x}_A-C\mathbf{x}_B,\mathbf{x}_B) =Cov(\mathbf{x}_A,\mathbf{x}_A)-C\cdot Cov(\mathbf{x}_B,\mathbf{x}_B)=0$$
$$ \Sigma_{AB} - C\Sigma_{BB} = 0 ;\Rightarrow\; C = \Sigma_{AB}\Sigma_{BB}^{-1} $$



**Step 2：條件均值。** $\mathbf{x}_A = \mathbf{z} + C\mathbf{x}_B$，條件於 $\mathbf{x}_B$ 時 $C\mathbf{x}_B$ 為常數：

$$ \mathbb{E}[\mathbf{x}_A \mid \mathbf{x}_B]=E[\mathbf{z}+C\cdot\mathbf{x}_B|\mathbf{x}_B] $$
$$ \mathbb{E}[\mathbf{z}] + C\mathbf{x}_B = (\boldsymbol{\mu}_A - C\boldsymbol{\mu}_B) + C\mathbf{x}_B  $$
$$== \boldsymbol{\mu}_A + \Sigma_{AB}\Sigma_{BB}^{-1}(\mathbf{x}_B - \boldsymbol{\mu}_B)$$

**Step 4：條件共變異數。**

$$ \operatorname{Var}(\mathbf{x}_A \mid \mathbf{x}_B)= \operatorname{Var}(\mathbf{z}+C\cdot\mathbf{x}_B \mid \mathbf{x}_B)  = \operatorname{Var}(\mathbf{z}) $$
$$= \Sigma_{AA} - \Sigma_{AB}C^\top - C\Sigma_{BA} + C\Sigma_{BB}C^\top $$

代入 $C = \Sigma_{AB}\Sigma_{BB}^{-1}$、$C^\top = \Sigma_{BB}^{-1}\Sigma_{BA}$，三個修正項都化為 $\Sigma_{AB}\Sigma_{BB}^{-1}\Sigma_{BA}$，係數為 $-1-1+1 = -1$：

$$ \Sigma_{A|B} = \Sigma_{AA} - \Sigma_{AB}\Sigma_{BB}^{-1}\Sigma_{BA} $$

---

## 推導二：對 pdf 指數項配方（精度矩陣版）

**Step 1：條件密度正比於聯合密度。**

$$ p(\mathbf{x}_A \mid \mathbf{x}_B) \propto p(\mathbf{x}_A, \mathbf{x}_B) \propto \exp!\left(-\tfrac{1}{2}J\right), \qquad J = (\mathbf{x} - \boldsymbol{\mu})^\top \Lambda (\mathbf{x} - \boldsymbol{\mu}) $$

其中精度矩陣

$$ \Lambda = \begin{bmatrix} \Lambda_{AA} & \Lambda_{AB} \\ \Lambda_{BA} & \Lambda_{BB} \end{bmatrix} = \begin{bmatrix} \Sigma_{AA} & \Sigma_{AB} \\ \Sigma_{BA} & \Sigma_{BB} \end{bmatrix}^{-1}, \qquad \Lambda_{BA} = \Lambda_{AB}^\top $$

$\mathbf{x}_B$ 已觀測，視為常數；將 $J$ 當作 $\mathbf{x}_A$ 的函數，只需對 $\mathbf{x}_A$ 配方。

**Step 2：展開 $J$。** 令 $\mathbf{y}_A = \mathbf{x}_A - \boldsymbol{\mu}_A$，$\mathbf{y}_B = \mathbf{x}_B - \boldsymbol{\mu}_B$：

$$ J = \mathbf{y}_A^\top \Lambda_{AA}\mathbf{y}_A + \mathbf{y}_A^\top \Lambda_{AB}\mathbf{y}_B + \mathbf{y}_B^\top \Lambda_{BA}\mathbf{y}_A + \mathbf{y}_B^\top \Lambda_{BB}\mathbf{y}_B $$

兩個交叉項互為轉置且為純量，故相等；最後一項與 $\mathbf{x}_A$ 無關：

$$ J = \mathbf{y}_A^\top \Lambda_{AA}\mathbf{y}_A + 2\mathbf{y}_A^\top \Lambda_{AB}\mathbf{y}_B + \text{const} \rightarrow J_1$$

按 $\mathbf{x}_A$ 的次數重排：

$$ J = \mathbf{x}_A^\top \Lambda_{AA}\mathbf{x}_A - 2\mathbf{x}_A^\top\big[\Lambda_{AA}\boldsymbol{\mu}_A - \Lambda_{AB}(\mathbf{x}_B - \boldsymbol{\mu}_B)\big] + \text{const} \rightarrow J_2$$

**Step 3：與高斯指數比較係數。**
- $\mathcal{N}(\boldsymbol{\mu}_{A|B}, \Sigma_{A|B})$ 的指數項展開為

$$ (\mathbf{x}_A - \boldsymbol{\mu}_{A|B})^\top \Sigma_{A|B}^{-1}(\mathbf{x}_A - \boldsymbol{\mu}_{A|B}) = \mathbf{x}_A^\top \Sigma_{A|B}^{-1}\mathbf{x}_A - 2,\mathbf{x}_A^\top \Sigma_{A|B}^{-1}\boldsymbol{\mu}_{A|B} + \text{const} $$

  
$$J_1=(\mathbf{x}_A - \boldsymbol{\mu}_{A\vert{}B})^T \boldsymbol{\Sigma}_{A\vert{}B}^{-1} (\mathbf{x}_A - \boldsymbol{\mu}_{A\vert{}B}) $$
$$J_2=\mathbf{x}_A^\top\boldsymbol{\Lambda}_{AA}\mathbf{x}_A-2\mathbf{x}_A^\top\left[\boldsymbol{\Lambda}_{AA}\boldsymbol{\mu}_A-\boldsymbol{\Lambda}_{AB}(\mathbf{x}_B-\boldsymbol{\mu}_B)\right]+\text{const}$$
$$(\mathbf{x}_A - \boldsymbol{\mu}_{A\vert{}B})^T \boldsymbol{\Sigma}_{A\vert{}B}^{-1} (\mathbf{x}_A - \boldsymbol{\mu}_{A\vert{}B}) = \mathbf{x}_A^T \boldsymbol{\Sigma}_{A\vert{}B}^{-1} \mathbf{x}_A - 2 \mathbf{x}_A^T \boldsymbol{\Sigma}_{A\vert{}B}^{-1} \boldsymbol{\mu}_{A\vert{}B} + \text{const}$$

- 比係數, 兩邊都對 $\mathbf{x}_A$ 微分兩次。

$$

\nabla_{\mathbf{x}_A}^2J_1=\nabla_{\mathbf{x}_A}(2\boldsymbol{\Sigma}_{A|B}^{-1}\mathbf{x}_A-2\boldsymbol{\Sigma}_{A|B}^{-1}\boldsymbol{\mu}_{A|B})=2\boldsymbol{\Sigma}_{A|B}^{-1}

$$
$$

\nabla_{\mathbf{x}A}^2J_2=\nabla_{\mathbf{x}_A}(2\boldsymbol{\Lambda}_{AA}\mathbf{x}_A-2\left[\boldsymbol{\Lambda}_{AA}\boldsymbol{\mu}_A-\boldsymbol{\Lambda}_{AB}(\mathbf{x}_B-\boldsymbol{\mu}_B)\right])=2\boldsymbol{\Lambda}_{AA}

$$

- 二次項（等價於兩邊對 $\mathbf{x}_A$ 取 Hessian）：

$$ \Sigma_{A|B}^{-1} = \Lambda_{AA} ;\Rightarrow; \Sigma_{A|B} = \Lambda_{AA}^{-1} $$

- 一次項（等價於比較梯度）：

$$ \Sigma_{A|B}^{-1}\boldsymbol{\mu}_{A|B} = \Lambda_{AA}\boldsymbol{\mu}_A - \Lambda_{AB}(\mathbf{x}_B - \boldsymbol{\mu}_B) $$

代入 $\Sigma_{A|B}^{-1} = \Lambda_{AA}$：

$$ \boldsymbol{\mu}_{A|B} = \boldsymbol{\mu}_A - \Lambda_{AA}^{-1}\Lambda_{AB}(\mathbf{x}_B - \boldsymbol{\mu}_B) $$

**Step 4：用分塊求逆（Schur complement）轉回 $\Sigma$。**

$$ \Lambda_{AA} = (\Sigma_{AA} - \Sigma_{AB}\Sigma_{BB}^{-1}\Sigma_{BA})^{-1}, \qquad \Lambda_{AA}^{-1}\Lambda_{AB} = -\Sigma_{AB}\Sigma_{BB}^{-1} $$

代回：

$$ \Sigma_{A|B} = \Sigma_{AA} - \Sigma_{AB}\Sigma_{BB}^{-1}\Sigma_{BA}, \qquad \boldsymbol{\mu}_{A|B} = \boldsymbol{\mu}_A + \Sigma_{AB}\Sigma_{BB}^{-1}(\mathbf{x}_B - \boldsymbol{\mu}_B) $$

