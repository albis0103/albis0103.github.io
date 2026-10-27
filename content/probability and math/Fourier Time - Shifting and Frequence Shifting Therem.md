
let $f(t) \xleftrightarrow{\mathcal{F}} F(\omega)$
$$f(t-t_0) \xleftrightarrow{\mathcal{F}} e^{-j\omega t_0}F(\omega)$$

**Proof**
$$\mathcal{F}\{f(t-t_0\} = \int_{-\infty}^{\infty}f(t - t_0)e^{-i\omega t}dt$$let $\tau = t-t_0$  , so $t = \tau + t_0$ , $dt = d\tau$ 
$$\int_{-\infty}^{\infty}f(\tau)e^{-i\omega (\tau+t_0)}d\tau=e^{-i\omega t_0}\int_{-\infty}^{\infty}f(\tau)e^{-i\omega \tau}d\tau = e^{-i\omega t_0}\mathcal{F}(\omega)$$ 
- 時域平移振幅($F(\omega)$的模長$|F(\omega)|$)不變: $|e^{-j\omega t_0}F(\omega)| = |F(\omega)|$
- 相位改變：$\angle F(\omega)$ 增加 $-\omega t_0$ 項
	- note: **相位**：點與座標平面的夾角
	- 相位差：發生時間延遲的訊號與原始訊號的同個點相位角差異![[Pasted image 20260919150932.png]]
