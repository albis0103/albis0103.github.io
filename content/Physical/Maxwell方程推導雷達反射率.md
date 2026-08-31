

介電小球（半徑 $a \ll \lambda$）置於外加電場 $E_0 e^{-i\omega t}$ 中，被極化成振盪偶極子。

**極化率**（由邊界條件 $\nabla \cdot \mathbf{D}=0$、$E_\parallel$ 連續解出）：
	均勻外場$E_0$(z軸)，介電球半徑$a$ 、相對介電常數$\varepsilon_r$ ，球外是真空（$\varepsilon_0$）
	電位$\Phi$兩區都滿足Laplace Equation($\nabla^2 \Phi - 0$)[[Laplace Equation]]

$$ \alpha = 4\pi\varepsilon_0 a^3 \cdot \frac{\varepsilon_r - 1}{\varepsilon_r + 2} $$

推導
- 球外($r<a$):Φ_out = −E₀r cosθ + (B/r²)cosθ (第一項是原本的均勻外場,第二項是球被極化後產生的偶極場修正)
- 球內(r<a):Φ_in = −E_in·r cosθ (球內電場必須有限,r→0 時不能發散,所以不能有 1/r² 項)

**兩個邊界條件,r=a 處**:

(1) 電位連續(否則電場會發散) $$-E_0a+\frac{B}{a^2}=-E_{in}a$$

(2) 法向 D 連續(無自由面電荷,∇·D=0 的邊界形式) 先算徑向電場 E_r=−∂Φ/∂r:

- 球外:E_r,out(a) = E₀ + 2B/a³
- 球內:E_r,in(a) = E_in

D 連續即 ε₀E_r,out = ε₀εᵣE_r,in,所以: $$E_0+\frac{2B}{a^3}=\varepsilon_rE_{in}$$

**解這兩個聯立方程**(兩個未知數 B、E_in): 由(1)得 E_in = E₀ − B/a³,代入(2):

$$E_0+\frac{2B}{a^3}=\varepsilon_r\left(E_0-\frac{B}{a^3}\right)$$

$$\frac{B}{a^3}(2+\varepsilon_r)=E_0(\varepsilon_r-1)$$

$$B=a^3E_0\cdot\frac{\varepsilon_r-1}{\varepsilon_r+2}$$

**連到偶極矩**:球外電位的第二項本來就是標準偶極場的定義形式 Φ_dipole = p·cosθ/(4πε₀r²),所以 B=p/(4πε₀),即:

$$p=4\pi\varepsilon_0B=4\pi\varepsilon_0a^3E_0\cdot\frac{\varepsilon_r-1}{\varepsilon_r+2}$$

因為 p=αE₀,兩邊消掉 E₀,就是文件裡的:

$$\alpha=4\pi\varepsilon_0a^3\cdot\frac{\varepsilon_r-1}{\varepsilon_r+2}$$

**這步的本質**:兩個邊界條件、兩個未知數,解線性方程組——跟你解任何邊界值問題的套路完全一樣,只是套用在電位方程上。






**感應偶極矩**：

$$ p = \alpha E_0 e^{-i\omega t} $$

**振盪偶極子輻射功率**（Larmor 公式，偶極輻射）：

$$ P_{rad} = \frac{\omega^4 |p|^2}{12\pi\varepsilon_0 c^3} $$

**入射波強度**：

$$ I_{inc} = \frac{1}{2}\varepsilon_0 c E_0^2 $$

**散射截面定義**：

$$ \sigma \equiv \frac{P_{rad}}{I_{inc}} = \frac{\omega^4 \alpha^2}{6\pi\varepsilon_0^2 c^4} = \frac{2\pi^5}{3}\cdot\frac{a^6}{\lambda^4}\left|\frac{\varepsilon_r-1}{\varepsilon_r+2}\right|^2 $$

代入 $D=2a$（直徑），並定義 $K = \dfrac{\varepsilon_r-1}{\varepsilon_r+2}$（複折射率因子），單一粒子的**後向**散射截面（雷達實際量測的是後向散射，不是全散射，係數略有差異，但比例關係相同）：

$$ \boxed{\sigma_i = \frac{\pi^5}{\lambda^4}|K|^2 D_i^6} $$

