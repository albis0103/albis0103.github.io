
描述黏性流體的運動,是質量守恆與動量守恆(牛頓第二定律)應用在流體上的結果。[[牛頓第二定律]]


$$\rho\left(\frac{\partial \mathbf{u}}{\partial t} + \mathbf{u}\cdot\nabla\mathbf{u}\right) = -\nabla p + \mu\nabla^2\mathbf{u} + \mathbf{f}$$

$$\nabla\cdot\mathbf{u} = 0$$
- $\mathbf{u}(\mathbf{x},t)$:速度場
左邊=$ma$ (質量* 加速度)
- $\rho$:密度
- - $\frac{\partial\mathbf{u}}{\partial t}+\mathbf{u}\cdot\nabla\mathbf{u}$: 流體的加速度$a$ 可以寫成寫成 $\dfrac{D\mathbf{u}}{Dt}$，（$\frac{D\mathbf{u}}{Dt} = \frac{\partial \mathbf{u}}{\partial t} + \mathbf{u}\cdot\nabla\mathbf{u}$$）因為速度場 $\mathbf{u}(\mathbf{x},t)$ 會因為「位置」跟「時間」改變
	- $\partial\mathbf{u}/\partial t$:局部加速度
	- $\mathbf{u}\cdot\nabla\mathbf{u}$:對流項(非線性,是難解的主因)，流體移動到了新的位置,而新位置本來的流速跟舊位置不一樣(比如水流進窄管,流速自動變快,染料滴即使沒有外力也會被迫加速)
右邊 = $F$（作用在流體上合力）
- - $-\nabla p$:壓力造成的力（壓力梯度驅動流動）
- - $\mu\nabla^2\mathbf{u}$:黏性摩插立造成的力（動量的擴散項,類似熱傳導方程)
	- - $\mu$:動力黏度係數
- - $\mathbf{f}$:外力(如重力)




**關鍵特性:**

- 非線性偏微分方程組,對流項 $\mathbf{u}\cdot\nabla\mathbf{u}$ 使其解析解幾乎不可能求得(除極簡單邊界條件外)
- 三維情況下解的存在性與光滑性是 Clay 研究所的千禧年七大難題之一,至今未解
- 數值解法(CFD)常用有限體積法、有限元素法,或譜方法離散化後求解

**無量綱化後的雷諾數** $Re = \rho U L/\mu$ 決定流動是層流還是紊流(湍流),是判斷對流項相對黏性項重要性的指標。
