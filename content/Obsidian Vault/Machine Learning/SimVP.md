SimVP(Simpler yer better Video Prediction): pure CNN、 MSE loss, to approximate multiple benchmark for SOTA, use Encoder, Translator, Decoder Module.
- **Faster Training Speed than ConvLSTM** 
	:Input, Output dimension $(T, C, H, W)$ separate the time axis , None recursive-based
- Native CNN: None of attention or recurrent unit
- **MSE** loss function: predict pixel is Regression
	$$\mathcal{L} = \frac{1}{T' \cdot C \cdot H \cdot W} \sum_{t=1}^{T'} \sum_{c=1}^{C} \sum_{i=1}^{H} \sum_{j=1}^{W} \left( \hat{Y}{t,c,i,j} - Y{t,c,i,j} \right)^2$$
$\mathbf{X}_{T \times C \times H \times W} \xrightarrow{\text{Encoder（Share Framed）}} \mathbf{Z}_{enc} \xrightarrow{\text{reshape Time→channel}} \xrightarrow{\text{Mid-Xnet}} \mathbf{Z}_{trans} \xrightarrow{\text{reshape channel→時間}} \xrightarrow{\text{Decoder（+skip from encoder）}} \hat{\mathbf{Y}}_{T \times C \times H \times W}$ 
![[Pasted image 20260812153810.png]]

- Spatial Encoder: Convolution each 2D-Frame to down-sampling
	- Joint Time dimension $T$ to channel dimension $C$  
		- $\mathbf{X}\in\mathbb{R}^{T\times C\times H\times W}\rightarrow f_{enc}(\mathbf{X}) \rightarrow \mathbf{Z} \in \mathbb{R}^{(T\cdot C)\times H'\times W'}, H'=\frac{H}{2^{N_s}}$ 
	- Structure
		- Spatial Convolution modules $\times N_s$ 
	- Down-sampling:
		- Kernel: $3\times 3$, Stride = $2, 1, 2, 1, \cdots$ for $N_s$ times
- Temporal Translator(mid-Xnet): U-Net architecture(Inception-like)
	- Input: $\mathbf{Z}_{enc} \in \mathbb{R}^{(T \cdot C_{hid}) \times H' \times W'}$
		- Consist of Multiple layer **Group Inception Module**
			- $\text{Inception}_{input}$: each Layer $1\times1 \text{ conv}$ justify channel $C$ concurrent each kernel ($3\times 3, 5\times 5, 7\times 7, ..etc$)
			- $\sum \text{each kernel output}$
			- $\text{skip connection}\rightarrow \text{Inception}_{output}$  
	- Output: $\mathbf{Z}_{trans} \in \mathbb{R}^{(T \cdot C_{hid}) \times H' \times W'}$ 
- Decoder
	- $\mathbf{Z}_{trans} \in \mathbb{R}^{(T \cdot C_{hid}) \times H' \times W'} \rightarrow f_{dec}(\mathbf{Z})\rightarrow \hat{ \mathbf{Y}}\in\mathbb{R}^{T\times C\times H\times W}$ 