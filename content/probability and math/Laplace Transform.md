$$\mathcal{L}\{f(t)\}=F(s)=\int_0^{\infty}e^{-st}f(t)dt$$
- $s$: complex number, $s=\sigma+iw$
	- $iw$: process imaginary part
	- $\sigma$: process exponential variance, real part


Common Transform

| $f(t)$                         | $F(s)$                         |
| ------------------------------ | ------------------------------ |
| $1$                            | $\frac{1}{s}$                  |
| $e^{at}$                       | $\frac{1}{s-a}$                |
| $\frac{t^{n-1}}{(n-1)!}e^{at}$ | $\frac{1}{(s-a)^n}$            |
| $sin(\omega t)$                | $\frac{\omega}{s^2+\omega ^2}$ |
| $cos(\omega t)$                | $\frac{s}{s^2+\omega ^2}$      |

Key Property
- Linear: $\mathcal{L}\{af(t)+bg(t)\}=aF(s)+bG(s)$
- Deviation:$\mathcal{L}\{f^{(n)}(t)\}=s^nF(s)-F(0)-\sum_{i=1}^ns^{n-1-i}f^{(i)}(0)$
	- $\mathcal{L}\{f'(t)\}=sF(s)-f(0)$
	- $\mathcal{L}\{f''()t\}=s^2F(s)-sF(0)-f(0)$
- Integral: $\mathcal{L}\{\int_0^tf(\gamma)d\gamma\}=\frac{F(s)}{s}$ 
- Convolution: $\mathcal{L}\{f*g\}=F(s)G(s)$ [[Convolution Theorem]]

given ODE resolve the $f(t)$
1. $F(s) = \frac{A}{s-p_1}+\frac{B}{s-p_2}+ \cdots$
2. by Residue method: 
	$A = Res(F, p1) = lim_{s \rightarrow p1}(s-p_1)F(s)$
		$=lim_{s \rightarrow p1}(s-p_1)\cdot F(s)$
	to get $A, B$
3. used $L^{-1}\{\frac{1}{s-a}\}=e^{at}$ , get $f(t)$
4. validation



### Example: 
 
 $$y''(t) + 3y'(t) + 2y(t) = 0, \quad y(0)=1,\ y'(0)=0$$

Resolve ODE function to get the orignal function $f(t)$

1. Laplace Transfrom
 $$\mathcal{L}{y''} = s^2Y(s) - sy(0) - y'(0)$$ $$\mathcal{L}{y'} = sY(s) - y(0)$$

	so Original function
	
	$$\big[s^2Y(s) - s(1) - 0\big] + 3\big[sY(s) - 1\big] + 2Y(s) = 0$$

2.  Organize Equation

$$s^2Y(s) - s + 3sY(s) - 3 + 2Y(s) = 0$$

$$(s^2+3s+2)Y(s) = s+3$$

$$Y(s) = \frac{s+3}{s^2+3s+2} = \frac{s+3}{(s+1)(s+2)}$$


3. Let $Y(s) = \frac{A}{s+1} + \frac{B}{s+2}$ , used residue theorem $A = lim_{s \rightarrow p_1}(s-p_1)F(s)$ to resolve $A, B$

 $$A = \lim_{s\to -1}(s-p_1)Y(s) = \lim_{s\to -1}(s-p_1)\cdot \frac{s+3}{(s+1)(s+2)}=\lim_{s\to -1}\frac{s+3}{s+2}$$
 $$\frac{-1+3}{-1+2} = \frac{2}{1} = 2$$
  
$$B = \lim_{s\to -2}Y(s) = \frac{-2+3}{-2+1} = \frac{1}{-1} = -1$$

$$Y(s) = \frac{2}{s+1} - \frac{1}{s+2}$$
4. Inverse Laplace equation

 $\mathcal{L}^{-1}{1/(s-a)} = e^{at}$：

$$y(t) = 2e^{-t} - e^{-2t}$$

5. validation

- $y(0) = 2-1 = 1$ ✓
- $y'(t) = -2e^{-t}+2e^{-2t}$，$y'(0) = -2+2 = 0$ ✓