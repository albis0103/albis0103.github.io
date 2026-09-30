Solid 
- 受到剪應力(Shear stress)會變形，一個固定變形量(deformation)抵抗剪應力，成線性關係
- 討論：應力 v.s.應變量(strain)
Fluid
- 變形量會一直增加不會停，停下表示shear stress == 0
- 流體內垂直排列的質點，隨時間會會變斜線
- 討論：應力 v.s. 應變量（strain rate/ deformation rate）
- 牛頓流體(Newtonian fluid)：Shear stress $\gamma$ 
- : Deformation rate 成線性關係
	$$\gamma = \mu \frac{du}{dy}$$
	即牛頓黏性定律(Newton's viscoity law)
	- $\mu$：絕對黏滯參數(Absolute viscosity) or 流體黏滯參數(viscosity) or 動力黏滯參數(dynamic viscosity)
	- $\frac{du}{dy}$:速度梯度（水平速度$u$在垂直$y$方向上變化）或角變率(rate of angular deformation)
	- $u$：流體流動方向之速度
	- $y$：垂直流動方向之速度

- 可以是混合物(mixture), or 相對流(multiphase)

在什麼條件可以把離散分子（$\rho, p, u$ ）當成練續場處理
- ANS: Continuum hypothesis
Continuum Hypothesis 連續性假設
：流體由分子組成，比較特徵長度：$\delta V^{\frac{1}{3}}$ 、平均自由長度(mean free path)

note：平均自由長度(Mean Free Path) $\lambda$：一個粒子在連續兩次碰撞能夠自由移動的距離
continuum 連續體: 隨空間與時間變化很平滑可以被微分
- 密度 $\rho=\frac{\delta N \cdot m}{\delta V}$ : 取很小的體積$\delta V$ 裡面的粒子數$\delta N$  
	![[Pasted image 20260724152323.png]]
case 1: 分子數相對少：
- $(\delta V)^{\frac{1}{3}} ≤ \text{mean free path}$ $\lambda$: 
	- $\delta V$ 太小,can not define $\rho$ : density，分子間距差不多，分子數會隨時間劇烈跳動（$t:$ 3 個分子，$t+1:$ 0 個），算出來的  ρ 根本不穩定
	- 需從氣體動力學 (gas dynamics) 或分子動力學 (molecular dynamics)描述
case2：分子數很多
![[Pasted image 20260901131247.png]]
- $(\delta V)^{\frac{1}{3}} > \text{mean free path}$ $\lambda$：
	- $\delta V$分子數遠大於mean free path
		- $\rho = \frac{\delta N}{\delta V} \cdot m$ (δV 內的分子數 * 分子密度)
	- 整個容器特徵長度$L$ 要遠大於特徵長度$\delta V^{\frac{1}{3}}$
- 當兩個條件都滿足 ($\lambda << (\delta V)^{\frac{1}{3}}<<L$)→ 流體可視為Continuum
	- $\rho = \rho(x, t)$ 

氣體動力學理論中平均自由路徑 $\text{mean free path} = \frac{1}{\sqrt{2}\pi f^2 n_v}$
 - $d$ = molecule diameter
 - $n_v$ = molecules per unit volume
 
 理想氣體方程式 :ideal gases  = $\frac{N_A P}{RT}$ 
  - $N_A$：亞佛加厥常數 (Avogadro's number)
  - $P$：壓力，
 -  $R$：氣體常數
  - $T$：溫度
 