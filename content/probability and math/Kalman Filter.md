Kalman-Filter is recursive-based Bayes-estimator, to estimate dynamic system state.

Assumption: Original System-noise(environment interference) and Measuring-noise $\in \text{Guassian Distribution}$ , and System $\in \text{linear}$ 
![[IMG_2978.jpg|399]]


System state:
- State - Transform: $x_k = A_kx_{k-1}+B_ku_k+w_k$
- Measure: $z=H_kx_k+v_k$

	- $x_k$:hidden state at time $k$ 
	- $z_k$ : observation at time $k$
	- $w_k \sim N(0, Q_k)$: system noise
	- $v_k \sim N(0, R_k)$: measuring noise

	- $u_k$:user input ex.user tuned radar parameter
	- $A_k, B_k, H_k$: is already known value Matrix
		- $A_k$:state transform matrix, describe system origin dynamic
			ex. state  $x= (\mathbf{p},\mathbf{v})^T=\begin{pmatrix}\text{position} \\ \text{velocity} \end{pmatrix}=\begin{pmatrix}1 & \Delta t \\ 0 & 1\end{pmatrix}$ 
		- $B_k$: Control matrix, describe already known external force
			if non-known external force, transform all from Original-environment($A_k$) and random noise($w_k$), $B=0$, $Bu_k=0$
		- $H_k$: Observation matrix, to let System-state and Observation well define, transform System-state structure $\rightarrow$ Observation structure
			ex.
			- GPS can not measure the velocity, $z \in \mathbf{R}^2$
				then $H=\begin{pmatrix}1 & 0\end{pmatrix}$ , let $z_k = Hx_k = \text{position}_k$ 

Algo:
- Predict
	$\hat{x}_{k|k-1}=A_k\hat{x}_{k-1|k-1}+B_ku_k$
	$P_{k|k-1}=A_kP_{k-1|k-1}A_k^T+Q_k$
		Covariance Matrix: 
		$\mathbf{P}_k=\mathbf{E}[(x_k-\hat{x}_k)(x_k-\hat{x}_k)^T]$: time $k$ Estimate Error Covariance
		$\mathbf{Q}_k=\mathbf{E}[w_kw_k^T]$: Process noise $w_k$ Covariance Matrix
		
			$\mathbf{P}_o=\begin{pmatrix}\sigma_{p_x}^2 & 0 & 0 & 0 \\ 0 & \sigma_{p_y}^2 & 0 & 0 \\ 0 & 0 & \sigma_{v_x}^2 & 0 \\ 0 & 0 & 0 & \sigma_{v_y}^2 &\end{pmatrix}, \mathbf{Q} = \begin{pmatrix} \sigma_{p_x}^{2} & \sigma_{p_x,p_y} & \sigma_{p_x,v_x} & \sigma_{p_x,v_y} \\\sigma_{p_y,p_x} & \sigma_{p_y}^{2} & \sigma_{p_y,v_x} & \sigma_{p_y,v_y} \\ \sigma_{v_x,p_x} &\sigma_{v_x,p_y} & \sigma_{v_x}^{2} & \sigma_{v_x,v_y} \\ \sigma_{v_y,p_x} & \sigma_{v_y,p_y} &\sigma_{v_y,v_x} & \sigma_{v_y}^{2} \end{pmatrix}$ 
			
		

- Update
	Calculate Kalman Gain( $K_k=\frac{\sigma_{estimate}^2}{\sigma_{estimate}^2+\sigma_{process}^2}=\frac{P_{k|k-1}}{P_{k|k-1}+R_k}$ , when$H=1$)
		![[IMG_2979.jpg|281]]
	$K_k=P_{k|k-1}H_K^T(H_kP_{k|k-1}H_k^T+R_k)^{-1}$
	$\hat{x}_{k|k}=\hat{x}_{k|k-1}+K_k(z_k-H_k\hat{x}_{k|k-1})$
	$P_k=(I-K_kH_k)P_{k|k-1}$
![](https://miro.medium.com/v2/resize:fit:1400/1*EKevbo7TcQk_n-dDoVJnSA.png)

[[Kalman-Filter Example-GPS]]
