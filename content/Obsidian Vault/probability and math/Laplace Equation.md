描述系統達到穩態 Steady-State的場（不隨時間變化的平衡狀態）

$u=u(x_1, \cdots, x_n)$ scaler function, $\Delta$ (or $\nabla^2$ ): Laplacian
**Laplace Equation**(or **Harmonic function**)

**笛卡爾坐標**
$$\Delta u = \nabla^2 u = \sum_{i=1}^n\frac{\partial^2 u}{\partial x_i^2}=0$$ 
For 2-Dimension:
	$$\frac{\partial^2 u}{\partial x^2}+\frac{\partial^2 u}{\partial y^2}=0$$
For 3-Dimension:
	$$\frac{\partial^2 u}{\partial x^2}+\frac{\partial^2 u}{\partial y^2}+\frac{\partial^2 u}{\partial z^2}=0$$

屬於橢圓形偏微分方程(elliptic PDE)，由二階線性一班形勢判斷
	$Au_{xx}+Bu_{xy}+Cu_{yy}+\cdots = 0$
	$B^2-4AC$:
	- $<0$：橢圓型
	- $=0$：拋物型
	- $>0$：雙曲型


**極座標**
$u(x, y) = u(rcos\theta, rsin\theta)$. 對$x, y$求二次偏導($u_{xx}+u_{yy}=u_rr+\frac{1}{r}u_r+\frac{1}{r^2}u_{\theta \theta}$)
$$\Delta u = \nabla^2 u = \frac{\partial^2 u}{\partial r^2}+\frac{1}{r}\frac{\partial u}{\partial r}+ \frac{1}{r^2}\frac{\partial^2 u}{\partial \theta^2}=0$$
$$=\frac{1}{r}\frac{\partial}{\partial r}(r\frac{\partial u}{\partial r})+\frac{1}{r^2}\frac{\partial^2 u}{\partial \theta^2}$$ 
**分離變數法**
$u(r, \theta)=R(r)\Theta(\theta)$, 帶入 Laplace除$R\Theta$, $\frac{r^2R''+rR'}{R}+\frac{\Theta''}{\Theta}=0$
	兩項只依賴$r, \theta$，必等於常數，設$\lambda$ 
		$r^2R''+rR'-\lambda R=0, \Theta''+\lambda \Theta = 0$ 


角度方程：$\theta$ 需滿足週期性 $\Theta(\theta)=\Theta(\theta + 2\pi)$，故$\lambda = n^2(n=0, 1, 2, \cdots)$ 
解:
	$\Theta_n(\theta)=A_ncos(n\theta)+B_nsin(n\theta)$ 
逕向方程：$r^2R''+rR'-\lambda R$為Euler-Cauchy方程，$R = r^m$帶入得$m^2=n^2$
- $n ≠ 0$：$R_n(r)=C_nr^n+D_nr^{-n}$
- $n = 0$：$R_0(r)=C_0+D_0lnr$
**完整通解**
	$$u(r, \theta) = (C_0+D_0lnr)+ \sum_{n=1}^{infty}[(C_nr^n+D_nr^{-n})cosn\theta+(E_nr^n+F_nr^{-n}sinn\theta)]$$


**各項**

| 項                                         | 從何而來                  | 物理意義                  |
| ----------------------------------------- | --------------------- | --------------------- |
| $C_0$                                     | $n=0$ 的 $R_0=$ 常數解    | 均勻背景值                 |
| $D_0\ln r$                                | $n=0$ 的第二解            | 點源（如二維電荷點源的位勢）        |
| $r^n\cos n\theta,\ r^n\sin n\theta$       | $n\neq0$，$R_n=r^n$    | 隨 $r$ 增大而增大的模態        |
| $r^{-n}\cos n\theta,\ r^{-n}\sin n\theta$ | $n\neq0$，$R_n=r^{-n}$ | 隨 $r$ 增大而衰減的模態（原點處發散） |



**沒有唯一解**

1. **圓盤內部 $r<a$（含原點）** 要求解在 $r=0$ 處有界 $\Rightarrow$ 捨去 $\ln r$ 與 $r^{-n}$ 項： $$u(r,\theta) = \frac{a_0}{2} + \sum_{n=1}^{\infty} r^n(a_n\cos n\theta + b_n\sin n\theta)$$
    
2. **圓外部區域 $r>a$（無窮遠處，通常要求 $u$ 有界或趨於 0）** 捨去 $r^n$ 與（視情況）$\ln r$ 項： $$u(r,\theta) = a_0 + \sum_{n=1}^{\infty} r^{-n}(a_n\cos n\theta + b_n\sin n\theta)$$
    
3. **環形區域 $b<r<a$（不含原點，也非無窮域）** 兩種模態都保留，即最上面的完整式： $$u(r,\theta) = (C_0+D_0\ln r) + \sum_{n=1}^{\infty}\Big[(C_n r^n+D_n r^{-n})\cos n\theta+(E_n r^n+F_n r^{-n})\sin n\theta\Big]$$
    

---

**係數**

$C_0,D_0,\dots$（或 $a_n,b_n$）由邊界上的 Fourier 展開決定。例如圓盤內部問題，邊界 $u(a,\theta)=f(\theta)$ 給定後：

$$a_n = \frac{1}{\pi a^n}\int_0^{2\pi}f(\theta)\cos n\theta,d\theta,\qquad b_n=\frac{1}{\pi a^n}\int_0^{2\pi}f(\theta)\sin n\theta,d\theta$$

環形區域則需要**兩組**邊界條件（內圈 $r=b$、外圈 $r=a$）聯立求解 $C_n,D_n$（及 $E_n,F_n$），是一個 $2\times2$ 線性方程組。
