
- **Standard LSTM Problem**: input $x$ is 1-D vector, 2-D CNN input need to flatten $\rightarrow$ 1-D vector

**ConvLSTM(Convolution LSTM)**
- Let LSTM not only Time Memory, can also retain **Spatial Structure Inofrmation**
- used Convolution substitute LSTM each FC [[LSTM]]
	- $W([h_{t-1}, x_t]) \rightarrow$  **$*$( convolution)** $\rightarrow W*[h_{t-1}, x_t]$ 
		- $f_t=\sigma(W_f * [h_{t-1}, x_t]+b_f)$
		- $i_t=\sigma(W_i * [h_{t-1},x_t]+b_i)$
		- $\tilde{C}=tanh(W_C*[h_{t-1}, x_t]+b_c)$
		- $C_t=f_t \odot C_{t-1}+i_t \odot \tilde{C_t}$
		- $o_t=\sigma(W_o*[h_{t-1},x_t]+b_i)$
		- $h_t=o_t \odot tanh(C_t)$
	- $i_t, h_t, C_t \in \mathbf{R}^3$ (channel * height * width)


![](https://miro.medium.com/v2/resize:fit:1400/0*XDwSSLr2_ZjboPiH.png)
	$\mathcal{X}_t, \mathcal{X}_{t+1}$: time $t, t+1 \text{ Input:}x_t, x_{t+1}$ 
		$\mathcal{H}_{t-1} \rightarrow \mathcal{H_t}$ : $W_h*h_{t-1}\text{ from }f_t, i_t, o_t, \tilde{C}_t$ 
		$\mathcal{X}_t \rightarrow \mathcal{H}_{t+1}$ : $W_x*x_t \text{ from }f_t, i_t, o_t, \tilde{C}_t$


![](https://miro.medium.com/v2/resize:fit:1400/0*nDwWBt-ft02-dvbZ.png)

**BiConvLSTM(Bidirectional ConvLSTM)**
: is ConvLSTM Bidirectional extension, used forward($t = 1 \rightarrow T$)and backward($t = T \rightarrow 1$), let each time output can get previous and future information

![[Pasted image 20260721165523.png]]

	 $f_t=\sigma(W_f * [h_{t-1}, x_t]+b_f)$
	 $i_t=\sigma(W_i * [h_{t-1},x_t]+b_i)$
	 $\tilde{C}=tanh(W_C*[h_{t-1}, x_t]+b_c)$
	 $C_t=f_t \odot C_{t-1}+i_t \odot \tilde{C_t}$
	 $o_t=\sigma(W_o*[h_{t-1},x_t]+b_i)$
	 $h_t=o_t \odot tanh(C_t)$