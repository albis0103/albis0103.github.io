
https://uedu.tw/statistics/a/markov-chains-intro

**Markov Property**: The **Memoryless** Random process. Future state only depends on current state, irrelevant with previous state.

**Markov Process**
:The  random process $\{X_t\}_{t≥0}$  that satisfy **Markov Process**.
$$P(X_{t+1}=x_{t+1}|X_t=x_t,X_{t-1}=x_{t-1} \cdots X_0 = x_0)=P(X_{t+1}=x_{t+1}|X_t=x_t)$$


**Transition Matrix (or stochastic matrix)**
: Describe the Markov Process 
- **Transition probability** : $P_{ij}^{(1)}=P(X_{t+1}=j|X_t=i)$ , $\sum_jP_{ij}=1$ 
	: Describe the Markov Process

- Row: $x_t$  \  Column: $x_{t+1}$
$$P =
\begin{pmatrix}
p_{11} & p_{12} & \cdots & p_{1n} \\
p_{21} & p_{22} & \cdots & p_{2n} \\
\vdots & \vdots & \ddots & \vdots \\
p_{n1} & p_{n2} & \cdots & p_{nn}
\end{pmatrix}$$
	- Non-Negative: $P_{ij}≥0, \forall i, j$ 
	- Column Normalization: $\sum_jP_{ij}=1$ 
	- **Stationary Distribution** $\pi$ : Column Vector $\mathbf{R}^{1*n}$ , time $t$ state After Transition to time $t+1$ still same probability $\pi_j=\sum_i\pi_iP_{ij}$   
		- The $\pi$ is also the eigenvector of $\lambda=1$ that satisfy $\pi P = 1\cdot \pi$ 
	-  $n$ steps Transformation: Chapman-Kolmogorov function
		$$P_{ij}^{(n)}=P(X_{t+n}=j|X_t=i) = (P^n)_{ij}$$
	- **Ergodic Theorem**
		: Each state $i$, After time $n \rightarrow \infty$ will converge to $\pi$ 
		$$lim_{n \rightarrow \infty}p_{ij}^{(n)}=\pi_j, \forall i$$
		- **irreducibility**: Each state mutually reachable
		- **aperiodicity**: Can return to self state for any transition steps, period=1
		