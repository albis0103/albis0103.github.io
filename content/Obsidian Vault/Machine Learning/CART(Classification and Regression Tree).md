CART is type of Decision-Tree, make prediction by 'splitting' on feature.

### Splitting
not use `if-else` logical rule, used **impurity**
- Classification
	- $K$: numbers of categories
	- $p_{i,k}$:$\frac{\text{numbers of node i samples} == \text{category k}}{\text{numbers of node i samples}}$ 

	- Gini Impurity: $\text{gini}=1-\sum_{k=1}^Kp_{i,k}^2$

	- Entropy : $\text{Entropy}=\sum_{k=1, \; p_{i,k}>0}^K\;p_{i,k}\;log_2\;p_{i,k}$
	- Cost function: $J(j,t_j)=\frac{m_{left}}{m_i}I_{left}+\frac{m_{right}}{m_i}I_{right}$ 
		- numbers of samples from node $i$ : $m_i = m_{left}+m_{right}$ 
		- feature $j$ , threshold $t_j$ , Impurity $I$ 
- Regression: 
	- square error: $\text{SE}=\sum_{j \in \text{node}_i}(y_i-\bar y_j)^2$ 
	- Cost function: $J(j,t_j)=\frac{m_{left}}{m_i}SE_{left}+\frac{m_{right}}{m_i}SE_{right}$ 