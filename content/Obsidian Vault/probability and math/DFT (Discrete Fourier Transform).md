### 1-D
The Sequence $x[n]$ length $N$ , $n = 0, .., N-1$ 
$DFT(x[n]):$
$$X[k] = \sum_{n = 0}^{N-1}x[n]e^{-j2\pi\frac{kn}{N}}, k=0, .., N-1$$
note: sample $k$ point in cycle $N$ :$2\pi f = 2\pi\frac{kn}{N}k$ 

$IDFT(X[k]):$ 
$$x[n] = \frac{1}{N} \sum_{k=0}^{N-1} X[k]\, e^{j 2\pi kn/N}$$

### 2-D
$M \times N$ image 2D - DFT
$$X[k_1, k_2] = \sum_{n_1=0}^{M-1}\sum_{n_2=0}^{N-1} x[n_1, n_2]\, e^{-j2\pi\left(\frac{k_1 n_1}{M} + \frac{k_2 n_2}{N}\right)}$$

$X[k]=a+bj$ 是複數，可用極座標表示：

$$X[k]=|X[k]|e^{j\phi[k]}=|X[k]|(\cos\phi[k]+j\sin\phi[k])$$

$a=|X[k]|\cos\phi[k]$、$b=|X[k]|\sin\phi[k]$，因此
- 振幅：$|X[k]|=\sqrt{a^2+b^2}$
- 相位：$\phi[k]=\operatorname{atan2}(b,a)$ 

為什麼要用 atan2： $\tan^{-1}(b/a)$ 的值域只有 $(-\pi/2,\pi/2)$，且 $(a,b)$ 與 $(-a,-b)$ 的比值相同，無法區分。例如 $X[k]=-2+2j$：$\tan^{-1}(2/-2)=-\pi/4$ 是錯的，$\operatorname{atan2}(2,-2)=3\pi/4$ 才正確。