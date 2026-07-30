：利用大氣原理、流體力學方程，透過數值方法預測大氣狀態（動量、質量、能量、水氣守恆）

原始方程組(Primitive Equation)
:描述大氣運動的方程
6 個未知數$u, v, \omega$(or w)$, T, q, p$(or$\Phi$)
- 動量方程(Navier - Stokes)
	:描述風場變化
	ex.等壓座標(p-coordinate)
	- x方向分量： 求解$u$
		$\frac{du}{dt}-fv = -\frac{\partial \Phi}{\partial x}$
	-  y方向分量： 求解$v$
		$\frac{du}{dt}-fv = -\frac{\partial \Phi}{\partial y}$
	- 全導數: $\frac{d}{dt}=\frac{\partial}{\partial t}+u\frac{\partial}{\partial x}+v\frac{\partial}{\partial y}+\omega\frac{\partial}{\partial p}$
		- $u, y$:水平風速的x(東向)與y(北向)分量
		- $f$：柯氏參數(Coriolis parameter)
			$f = 2\Omega sin \phi$  
		- $\Phi$: 位勢(Geopotential), $\Phi(h)=\int_0^hg(\phi, z)dz$ 
			- $g(\phi, z)$:重力加速度
				- $\phi$：緯度
				- $z$：幾何高度
- 靜力方程：垂直方向動量方程簡化成靜力平衡  [[靜力方程（靜力平衡方程）]]
	$\frac{\partial \Phi}{\partial p}=-\frac{RT}{p}$ 
- 連續方程（等壓座標下）:質量守恆
	$\frac{\partial u}{\partial x}+\frac{\partial v}{\partial y}+\frac{\partial \omega}{\partial p}=0$ 
- 熱力學方程：描述溫度隨時間變化[[熱力學第一定律]]
	$\frac{d T}{dt}=\frac{\omega \alpha}{c_p}+\frac{Q}{c_p}$ 
	- $Q$：非隔熱加熱綠（淺熱釋放、輻射）
- 水氣方程：描述濕度與相變
	$\frac{dq}{dt} = E-C$
	- $E$：蒸發
	- $C$：凝結
- 理想氣體狀態方程
	$p = \rho RT$ 
	or
	$p\alpha = RT$, $\alpha=\frac{1}{p}$為比容


### 離散化+微分運算子換成代數運算+向前遞推
- 空間離散化：連續場$u(x, y, p, t)$換成離散值$u^n_{i, j, k}$ 
	- 有限差方法(Finite Difference)：：用差商近似微分。例如中央差分近似一階導數：
		$\frac{\partial u}{\partial x}\bigg|_{i} \approx \frac{u_{i+1} - u_{i-1}}{2\Delta x}$
	- 譜方法(Spectral Method)：把水平場展開成球諧函數(Spherical harmonics)
		$u(\lambda, \phi) = \sum_{n} \sum_{m} u_n^m Y_n^m(\lambda, \phi)$
nics）的線性組合：
- 時間離散化：跳蛙法(Leapfrog schema)
	$u^{n+1} = u^{n-1} + 2\Delta t \cdot F(u^n)$
	- 穩定條件
		$\Delta t \leq \frac{\Delta x}{c}$
