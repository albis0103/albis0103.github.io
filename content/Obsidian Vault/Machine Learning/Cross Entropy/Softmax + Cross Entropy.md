### why used CrossEntropy for classification

**MSE Gradient Vanish problem**
:$(a-y)a(1-a)$,when a → 0 or 1, $\frac{\partial L}{\partial z}$ → 0, $\Delta g \rightarrow 0$  

(activation function :sigmoid $\sigma(z)$, output: $a$)
note: $\sigma'(x)=\sigma(x)(1-\sigma(x))$

$L=\frac{1}{2}(y-a)^2$
$\frac{\partial L}{\partial z}=\frac{\partial L}{\partial a}\frac{\partial a}{\partial z}$
$\frac{\partial L}{\partial a}=-(y-a)=a-y$
$\frac{\partial a}{\partial z}=\sigma'(z)=\sigma(z)(1- \sigma(z)) = a(1-a)$
$\frac{\partial L}{\partial z}=(a-y)a(1-a)$



### Cross Entropy
: $a-y$, when $a \rightarrow$ 0 or 1, not inference  

$L=-[ylna+(1-y)ln(1-a)]$
$\frac{\partial L}{\partial a}=-\frac{y}{a}+\frac{1-y}{1-a}=\frac{a-y}{a(1-a)}$
$\frac{\partial a}{\partial z}=\sigma'(z)=\sigma(z)(1- \sigma(z)) = a(1-a)$
$\frac{\partial L}{\partial z}=\frac{a-y}{a(1-a)}a(1-a)=a-y$

$z_j$: softmax output(logit)

$\hat{y_j}=softmax(z_j)=\frac{e^{z_j}}{\sum_m e^{z_m}}$

$L=-\sum_k y_klog\hat{y_k}$:one-hot

$\frac{\partial L}{\partial z_i}$=?

1. $\frac{\partial L}{\partial z_i}=\sum_k \frac{\partial L}{\partial \hat{y_k}}\frac{\partial \hat{y_k}}{\partial z_i}$
    
2. $\frac{\partial L}{\partial \hat{y_k}}=- \frac{\partial }{\partial \hat{y_k}}\sum_ky_klog\hat{y_k}=-\frac{y_k}{\hat{y_k}}$
    
    （對 $\hat{y_k}$微 只留第k項, so: $-y_k\frac{\partial}{\partial \hat{y_k}}log\hat{y_k} = -\frac{y_k}{\hat{y_k}}$)
    
3. $\frac{\partial \hat{y_k}}{\partial z_i}=?$
    
    另 $S=\sum_me^{z_m}, \hat{y_k}=\frac{e^{z_k}}{S}$(note: by 商 微分 $\frac{\partial}{\partial z_i}(\frac{f}{g})=\frac{f'g-fg'}{g^2}$)
    
    1. if k = i, $f=e^{z_i}$ ( $z_i$ ↔ self output $\hat{y_i}$)
        
        $\frac{\partial \hat{y_k}}{\partial z_i}=\frac{e^{z_i}*S-e^{z_i}e^{z_i}}{S^2}$(對 $z_i$微 只留 $e^{z_i}$)
        
        $=\frac{S-e^{z_i}}{S}\frac{e^{z_i}}{S}=\hat{y_i}(1-\hat{y_i})$
        
    2. if k ≠ i, $f=e^{z_k}$ ( $z_i$ ↔ other output $\hat{y_k}$)
        
        $\frac{\partial \hat{y_k}}{\partial z_i}=\frac{0*S-e^{z_k}e^{z_i}}{S^2}\\ =-\frac{e^{z_k}}{S}\frac{e^{z_i}}{S}=-\hat{y_k}\hat{y_i}$
        
4. 帶入chain rule
    
    $\frac{\partial L}{\partial z_i}=-\sum_k \frac{y_k}{\hat{y_k}}\frac{\partial \hat{y_k}}{\partial z_i}$
    $=-\frac{y_i}{\hat{y_i}}\hat{y_i}(1-\hat{y_i})+\sum_{k≠i}\frac{y_k}{\hat{y_k}}\hat{y_k}\hat{y_i}$
    $\\ =-y_i(1-\hat{y_i})+\sum_{k≠i}y_k\hat{y_i}$
    $\\=-y_i+y_i\hat{y_i}+\hat{y_i}\sum_{k≠i}y_k$
    $\\=-y_i+\hat{y_i}(y_i+\sum_{k≠i}y_k)\\=\hat{y_i}-y_i$
    