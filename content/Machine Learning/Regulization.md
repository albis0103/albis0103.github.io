### Loss function
$$R(\theta) = \frac{1}{n}\sum_{i=1}^nL(y_i, f(x_i))$$
$$\theta^* = argminR(\theta) + \lambda\Omega(\theta)$$
Objective:
Avoid the **Variance - bias - trade-off**
![[Pasted image 20260922172414.png]]
$$MSE(f) = Bias(f)^2+Var(f)^2$$
### L1 (Lasso) 
$$\Omega(\theta) = ||\theta||_1=\sum_j|\theta|$$

### L2(Ridge)
$$\Omega(\theta) = ||\theta||_2^2 = \sum_j\theta_j^2$$