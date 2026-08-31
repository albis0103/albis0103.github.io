- Fluid v.s. solid
- 連續體 continuum：不可切割
	- local thermodynamic equilibrium：局部熱力平衡
		- 當一系統達到機械平衡，會有會有均勻的壓力存在
		- 達到熱平衡，會有均勻的溫度存在
		- 流力、熱傳學 屬於非平衡科學，透過局部熱力平衡假設，定義局部壓力跟局部溫度
- Streamlines流線、pathlines路徑線、streakline煙線、material lines質量線
	- 給一流場畫出這些線
- Fluid motion流體流動時受到的應力、應變 stress and strain rate
- 因式分析Dimensional Analysis: buckingham pi theorem定理
	- 將問題總體性整理得到關鍵性參數
- Dimensionless parameter

Solid
	對其師剪應力物理會變形，放開彈回，用一個變形量抵抗剪應力，奪大的剪應力shear street對應奪大的變形量
	討論應力與應變量
	沒有變形率deformation rate
Fluid
	變形量會一直增加，不會停，停下表示shear street = 0
	垂直線的值點partiacles隨時間增加便斜線
	討論應力與單位時間應變量
	因流體不同應變率不同
	牛頓流體Newtonian：shear stress v.s. deformation rate == linear relation ex.水、空氣
	不一定是純物質，可以是混合物ex.maxture ex.air or multiphase ex.water

continuum 連續體
	隨空間與時間變化很平話可以被微分
	![[Pasted image 20260724152323.png]]
	

$\text{in } \delta v_1, \delta v_2$ probability not same
	case1, 流體分子相對不多
		$(\delta v)^{\frac{1}{3}}$ ：特徵長度 與 mean free path平均自由路徑：分子分子平均碰撞距離 比較
			$(\delta v)^{\frac{1}{3}} ≤ \text{mean free path}$ ：can not define $\rho$: density
			需從氣體動力學 has synamic或粒子動力學molecular dynamics
	case2, 氣體分子很多
		$(\delta v)^{\frac{1}{3}} > \text{mean free path}$ ：可以算$\delta v$ 內粒子束的平均值 >> 分子跳動量fluctuation
		$\rho$ : $\delta V(x)$, when $\delta V$小到像一個點
		得到一個密度場 -> fuild is continuum
		-> well defined $\rho (x, t)$密度場為時間與空間的一個函數
		![[Pasted image 20260724154308.png]]
		$\rho = \frac{\delta N}{\delta V} \cdot m$ ：密度為 小體積中分子數目 *分子質量(m)
		當在區域$\delta V$ 特徵長度$\delta V^{\frac{1}{3}}$遠大於  平均自由路徑
		在整個容器 特徵長度 >> $\delta V$特徵長度$\delta V^{\frac{1}{3}}$時
		-> 連續體 continuum -> 流體動力學定義密度
		氣體動力學理論 Kinetic theory
		：定義定義莉子平均自由路徑
		$\text{mean free path} = \frac{1}{\sqrt{2}\pi f^2 n_v}$ , $d$ = molecule diameter, $n_v$ = molecules per unit volume
		理想氣體方程式 ideal gases = $\frac{N_A P}{RT}$, $N_A$：雅柏加爵常數, $P$：壓力，$R$：氣體常數，$T$：溫度
		
	