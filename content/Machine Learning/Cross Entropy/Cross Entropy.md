### Information(資訊量/驚訝度）

shannon coding

$I_p(x) = -log(x), p(x)$低, $I_p$高：罕見 、意外

### Entropy

$H(P) = E_{x\sim P}[-logP(x)]$

pf:

[[Lagrange Multiplier for Entropy]]

### note

K categories

- $\hat{y_{ij}}$：樣本 $x_i \in j$ 類的機率
    
- $y_i$： $x_i$真實類別tag
    
    $\hat{y_{ij}}=(\hat{y_{i1}}, \hat{y_{i2}}, .., \hat{y_{iK}}), \hat{y_{ij}}:softmax=\frac{e^{z_{ij}}}{\sum_{k=1}^k e^{z_{ik}}}$
    $e_{yi}^T \in R^k=\begin{bmatrix}0\ \vdots \\ 1 \\\vdots \\ 0\end{bmatrix}$ 代表真實標籤, ont hot 向量
    
    $\hat{y_{i, y_i}} = e^T_{yi}\hat{y_i} \\ \hat{y_{i, y_2}}= \begin{bmatrix}0&1&0 \end{bmatrix}\begin{bmatrix} y_{i1} \\ y_{i2} \\ y_{i3} \end{bmatrix}$
    

$q(y_i=j|x_i)$: reality distribution == $\hat{y_{ij}}$

$log(q(y_i|x_i))=log(\hat{y_{iy_i}})=\sum_{j=1}^K 1(y_i=j)*log(\hat{y_{ij}})$

### Cross Entropy

: 用 q(x)編碼，按p(x)的頻率出現，平均每個事件的bit數

$H(p,q)=E_{X\sim p}[-log(q_x)]=-\sum_i p_{x_i}log(q_{x_i})$

**for ML loss function**

1. $L(\theta):-\sum y_ilog\hat{y_i}$

because

[[MLE for Cross Entropy]]

Chain rule: $\frac{\partial L}{\partial w}=\frac{\partial L}{\partial \hat{y}}\frac{\partial \hat{y}}{\partial z}\frac{\partial z}{\partial w}, \hat{y}=softmax(z), z=wx+b$

1. $\frac{\partial L}{\partial z_k}=\hat{y_k}-y_k$, to avoid gradient vanish
    
    [[Softmax + Cross Entropy]]
    
2. $w \leftarrow w-\eta \frac{\partial L}{\partial w}$
    

### KL Divergence

:evaluate error arise from approximate distribution q → distribution p

$D_{KL}(P||Q)=H(p,q)-H(p)\\=E_{X\sim P}[log(q(x))-log(p(x))]\\=\sum_XP(x)log(\frac{P(x)}{Q(x)})$

[[Softmax + Cross Entropy]]