ML:

- regression
    [[Regression]]
    
    
- classification
    

ML Training 3 step

1. Function with unknown param
    1. find weight, bias → y = b + wx
2. Define Loss fuction from Training data
    1. $Loss(bias, weight)$→ evaluate value of set of value
        
    2. $err = |y - \hat{y}|$, $\hat{y}$: lable
        
        $$  
        L = \frac{1}{N} \sum_n err_n  
        $$
        
        1. MAE(mean aboulute error)= $|y_i - \hat{y}|$
        2. MSE(mean square error) = $(y_i-\hat{y})^2$
        3. cross-entropy : $- \sum_i y_ilog\hat{y_i}$
    3. Optimization
        $w^*, b^*=argmin_{w,b}L$
        1. Gradient Descent(local minima problem)
            1. random init value w_0
            2. compute w slope
![[Pasted image 20260715115214.png]]
1. Function with Unknown param

function justification: consider by question comprehension

$$  
y=b+wx_1 \\ \rightarrow y=b+\sum_{j=1}^{7}w_jx_j  
$$

reality world problem most of not represent by linear model

model bias: linear model have limination(note:model bias ≠ bias)

All piecewise Linear curves = constant + sum of set of plot
![[Pasted image 20260715115318.png]]
Beyond Piecewise Linear : can also approximate continume by piecewise linear curve(# node → infinite)![[Pasted image 20260715115354.png]]Sigmoid can represent Hard sigmoid
![[Pasted image 20260715115412.png]]
$$  
c*sigmoid(b,wx_1) = c * \frac{1}{1+e^{-(b+wx_1)}}  
$$

justify b,w,c to get different shape of curve of sigmoid

w:slope, b:shift, c:height
![[Pasted image 20260715115435.png]]
summation to get target curve

$y =b+wx_1\rightarrow b+\sum_{i}c_isigmoid(b_i+w_ix_1)$
$y=b+\sum_{j}w_jx_j\rightarrow b+\sum_{i}c_isigmoid(b_i+\sum_{j}w_{ij}x_j)$ 

Sigmoid → ReLu(Rectified Linear Unit):

$max(0, b+wx_1)$
![[Pasted image 20260715115506.png]]
Activation functinon ex.sigmoid, Relu

_i = 1,2,3 ,_

_j = 1,2,3_



![截圖 2026-06-09 晚上9.46.07.png](attachment:96ccf3e4-a917-42fc-b8fd-a17d6ff43c2f:%E6%88%AA%E5%9C%96_2026-06-09_%E6%99%9A%E4%B8%8A9.46.07.png)

$$  
\begin{bmatrix} r_1 \\ r_2 \\ r_3 \end{bmatrix}={\color{green}\begin{bmatrix} b_1 \\ b_2 \\ b_3 \end{bmatrix}}+\begin{bmatrix}w_{11} & w_{12} & w_{13} \\w_{21} & w_{22} & w_{23} \\w_{31} & w_{32} & w_{33}\end{bmatrix}\begin{bmatrix} x_1 \\ x_2 \\ x_3 \end{bmatrix}  
$$

then sigmoid(ri)

![截圖 2026-06-09 晚上10.13.28.png](attachment:8349ccef-d0ef-48fb-a0c0-c08319215767:%E6%88%AA%E5%9C%96_2026-06-09_%E6%99%9A%E4%B8%8A10.13.28.png)

![截圖 2026-06-09 晚上10.16.07.png](attachment:0e87faf2-bc70-4b36-bf36-8f687c483795:%E6%88%AA%E5%9C%96_2026-06-09_%E6%99%9A%E4%B8%8A10.16.07.png)

theta = all Unknown parameter

![截圖 2026-06-09 晚上10.18.31.png](attachment:6149b7c2-bfa7-42df-baf0-0be4e9e1fb3b:%E6%88%AA%E5%9C%96_2026-06-09_%E6%99%9A%E4%B8%8A10.18.31.png)

1. Loss function:L(theta)

![截圖 2026-06-09 晚上10.22.54.png](attachment:deb114f9-17c3-4d44-885d-82dc2f89cd47:%E6%88%AA%E5%9C%96_2026-06-09_%E6%99%9A%E4%B8%8A10.22.54.png)

![截圖 2026-06-09 晚上10.25.11.png](attachment:e0caa532-13b3-4cc9-94c2-c6643c39f7f6:%E6%88%AA%E5%9C%96_2026-06-09_%E6%99%9A%E4%B8%8A10.25.11.png)

$\theta^*=argmin_\theta L \leftarrow \theta^n - \eta \nabla L(\theta^n))$

$\dfrac{\partial L}{\partial \theta_1} \Big|_{\theta = \theta^0}$

1. Random initial value $\theta_0 = \begin{bmatrix} \theta^0_1 \\ \theta^0_2 \\ \vdots \end {bmatrix}, \theta_1 = \theta^0_1, \theta_2 = \theta^0_2, \dots$
    
2. compute gradient,
    
    $g=\nabla L(\theta^0)\\\theta^1 \leftarrow \theta^0 - \eta g$
    
3. compute gradient
    
    $g=\nabla L(\theta^1)\\\theta^2 \leftarrow \theta^1 - \eta g$…
    

but computing so much(for all $\theta^0_1 \dots \theta^0_n$) so let batch to Unit(ex.32( $\theta^0_1 \dots \theta^0_{32}$), 64, 128…)

![截圖 2026-06-09 晚上10.35.55.png](attachment:fa4a7138-b911-47ef-9784-694f95b115b2:%E6%88%AA%E5%9C%96_2026-06-09_%E6%99%9A%E4%B8%8A10.35.55.png)

for batch unit

- compute gradient :update
- see all batches onces: epoch

ex. N = 10000 , B = 10, epoch? ans:1000 update

**shuffle**: each epoch random allocate batch

|Full Batch|Mini Batch|
|---|---|
|Batch size||
|:1( all scaler == non batch)|Batchs size|
|:N(batch size)||
|Long time|short time|
|more powerful|more noise|

but noisy update some time will more precise, because Optimization Fails

the Flat Minima in local minima is good

![截圖 2026-06-14 晚上11.09.47.png](attachment:3832a515-7397-4382-8869-304842fd8147:%E6%88%AA%E5%9C%96_2026-06-14_%E6%99%9A%E4%B8%8A11.09.47.png)

**Momentum**

**: consider physical word put to Gradient descent, $\lambda$:hyperparamter, 決定前一步動量保留多少**

start at $\theta^0$, Movement $m^0=0$

$m^{n+1} \leftarrow \lambda m^n - \eta \nabla L( \theta^n),\\ \theta^{n+1} \leftarrow \theta^n + m^{n+1}$

large sigmoid or ReLu can construct any complex function curve, **but** it will cause overfitting

to justify the model precision

1. check training data loss:compare different model on traning and testing loss
    1. model bias
        1. model too simple : sol: add feature
    2. optimization : local minimum
2. check testing data loss
    1. overfitting
    2. mismatch

![截圖 2026-06-10 晚上11.19.49.png](attachment:2ce102ef-f910-41d0-8cab-01f0c79cb1dc:%E6%88%AA%E5%9C%96_2026-06-10_%E6%99%9A%E4%B8%8A11.19.49.png)

so how to train good model

![截圖 2026-06-10 晚上11.29.40.png](attachment:e1dfc983-3354-47fb-b2cc-bbca583a56d7:%E6%88%AA%E5%9C%96_2026-06-10_%E6%99%9A%E4%B8%8A11.29.40.png)

use N-fild Cross validation set to split data

: split N-subset 2 training set 1 validation set, and for each model evaluate avg MSE

![截圖 2026-06-10 晚上11.34.50.png](attachment:5eee76f1-d6b2-429d-badc-bcedcbde8ef0:%E6%88%AA%E5%9C%96_2026-06-10_%E6%99%9A%E4%B8%8A11.34.50.png)

Optimization Fail

- Gradient close to zero: critical point (ex. local minimum, saddle point)
    
    ![截圖 2026-06-11 晚上9.23.22.png](attachment:32b0447e-11e6-4cc9-82b6-19d02c34af8a:%E6%88%AA%E5%9C%96_2026-06-11_%E6%99%9A%E4%B8%8A9.23.22.png)
    

by

[Taylor Series](https://app.notion.com/p/Taylor-Series-3930596b3a418031bcafc09879e5b3ff?pvs=21)

$note:f(x)=\sum_{n=1}^\infin\frac{f^{(n)}(a)}{n!}(x-a)$

給定某組loss function param: $\theta '$

g: vector, $\nabla L(\theta), g_i=\frac{\partial L(\theta ')}{\partial \theta_i}$

Hessian H: matrix

![截圖 2026-06-11 晚上9.43.14.png](attachment:00da42f9-dffc-486c-b16b-36e54b259784:%E6%88%AA%E5%9C%96_2026-06-11_%E6%99%9A%E4%B8%8A9.43.14.png)

when at critical point gradient = 0

so focus on $\frac{1}{2}(\theta-\theta ')^TH(\theta - \theta ')$

$L(\theta ) = L(\theta ') + v^THv$(v = $\theta-\theta'$)

For all v: $v^THv>0$ → **local minimum**

For all v: $v^THv<0$ → **local maximum**

sometime $v^THv>0$, sometime $v^THv<0$then → **saddle point**

$u$ : eigenvector of H, $\lambda$ : eigenvalue of $u$

（代入 $Hv = \lambda v$）→ $u^THu=u^T(\lambda u)=\lambda||u||^2$

saddle point sol: follow eigenvector $u$ of negative eigenvalue $\lambda$

![截圖 2026-06-11 晚上9.54.09.png](attachment:bfbb984d-9645-4606-9d09-41b46ccda99e:%E6%88%AA%E5%9C%96_2026-06-11_%E6%99%9A%E4%B8%8A9.54.09.png)

$\mathbf{H}(f) =\begin{pmatrix}\dfrac{\partial^2 f}{\partial x^2} & \dfrac{\partial^2 f}{\partial x \partial y} \\[2ex]\dfrac{\partial^2 f}{\partial y \partial x} & \dfrac{\partial^2 f}{\partial y^2}\end{pmatrix}$

$Minimum ratio = \frac{\#of\quad Positive\quad EigenValue}{\# of \quad EigenValue}$

### Batch

: 不是對所有data微分 是以batch為單位

### Adaptive Learning Rate

: given learning rate for each parameter

- In General, low loss → low norm of gradient
    
    :when low loss , only less (<< $\theta.norm$) Top eigenvector
    

Training Stuck ≠ small Gradient: Low loss, but **Not Absolutly low norm of gradient**

![image.png](attachment:80cb6b1c-87d8-442c-8750-57d0a45e3ffd:image.png)

How to Customization Learning Rate per Parameter

For one parameter(only consider $\theta_i$):

Orig:

$\theta_i^{t+1} \leftarrow \theta_i^{t} - \eta g_i^t \\g_i^t = \frac{\partial L}{\partial \theta_i}\bigg|_{\theta = \theta^t}$表示對 $\theta_i$偏微後，入第t步 $\theta$ == $\frac{\partial L(\theta^t)}{\partial \theta_i}$

Customer Learning rage:

$\theta_i^{t+1} \leftarrow \theta_i^{t} - \frac{\eta}{\sigma^t_i} g_i^t$, $\sigma^t_i$: depend on $\theta_i$

Method:

- Root Mean Square: $\theta_i^{t+1} \leftarrow \theta_i^t - \frac{\eta}{\sigma_i^t} \cdot g_i^t, \sigma_i^t=\sqrt{\mathbb{E}[g_i^2]}=\sqrt{\frac{1}{t+1}\sum_{i=0}^t(g_i^t)^2}$
    
    $\theta^1_i \leftarrow \theta_i^0 - \frac{\eta}{\sigma^0_i}g_i^0, \sigma_i^0=\sqrt{(g^0_i)^2}=|g^0_i| \\  
    \theta^2_i \leftarrow \theta_i^1 - \frac{\eta}{\sigma^1_i}g_i^1, \sigma_i^0=\sqrt{\frac{1}{2}[(g^0_i)^2+(g^1_i)^2]}$
    
    ![image.png](attachment:3781229e-ccf8-4241-8c8e-360856e961c8:image.png)
    
    ![image.png](attachment:793043dd-e4e6-439d-8d54-8de69741145c:2cc5f7c5-52b4-4bae-9edf-7718e2350b85.png)
    

- RMSProp(Propagation): $\alpha$ =(0, 1): hyperparameter , used to signifitance of different varriable time parameter.  
    The recent gradient has larger influence, the past gradient has less influence.
    
    $\theta^1_i \leftarrow \theta_i^0 - \frac{\eta}{\sigma^0_i}g_i^0, \sigma_i^0=\sqrt{(g^0_i)^2}=|g^0_i| \\  
    \theta^2_i \leftarrow \theta_i^1 - \frac{\eta}{\sigma^1_i}g_i^1, \sigma_i^0=\sqrt{\alpha[(g^0_i)^2+(1-\alpha)(g^1_i)^2]}$
    
    ![image.png](attachment:84541838-18f1-4173-a291-37d0e35a5a76:image.png)
    
- Adam: RMSProp + Momentum
    
    :
    
    $\theta_i^{t+1} \leftarrow \theta_i^t - \frac{\eta}{\sigma_i^t} \cdot m_i^t$
    
    $\sigma_i^t=\sqrt{\alpha[(g^{t-1}_i)^2+(1-\alpha)(g^t_i)^2]}$
    
    $m_i^t = \beta m_i^{t-1} + (1- \beta)g_i^t$
    

Learning rate scheduling : Justify $\eta^t$ method

- Learning Rate Decay
    
    ![image.png](attachment:64729fd2-410b-4028-8348-1a8e01f4d90d:image.png)
    
- warm up : From Residual Network(used in transformer optimizer)
    
    explain: $\sigma = =\sqrt{\mathbb{E}[g^2]}$ 統計後結果，需大量資料。所以start先給他學習
    
    ![image.png](attachment:cefaf056-f25b-4abc-9f93-6ec356c78375:image.png)
    

### Summary of Optimization

- Gradient Desent
    
    $\theta_i^{t+1} \leftarrow \theta_i^t - \eta g_i^t$
    
- Various Improvement
    
    $\theta_i^{t+1} \leftarrow \theta_i^t - \frac{\eta ^t}{\sigma_i^t} \cdot m_i^t$
    
    $m_i^t$: Weighted sum of previous gradient(**Consider Direction**)
    
    $\sigma _i^t$: RMS of gradient(**Consider magnitude**)
    
    $\eta^t$: Learning rate scheduling
    

### Classification

- classification as regression
    
    : used one-hot vector to represent class
    
    avoid confuse relation between classs
    
    ex. 1: [1, 0, 0], 2:[0, 1, 0],, 3: [0, 0, 1] get Norm() between each is same
    
    ![image.png](attachment:25851474-aec5-4b96-a83c-47d7653bf880:image.png)
    

softmax (class > 2)→ input logit , let output in [0, 1]

softmax(z)_i = $\frac{e^{z_i}}{\sum_{j=1}^K e^{z_j}}$

sigmoid(class == 2)

$\sigma(z) = \frac{1}{1+e^{-z}}$

when K = 2

$P(y=1)=\frac{e_{z_1}}{e^{z_1}+e^{z_2}}  =\frac{e_{z_1}/e_{z_1}}{e^{z_1}/e_{z_1}+e^{z_2}/e_{z_1}}  =\frac{1}{1+e^{-(z_1+z_2)}}=\sigma(z_1-z_2)$

[Cross Entropy](https://app.notion.com/p/Cross-Entropy-3830596b3a41800c87adea6b9baff1c3?pvs=21)

### Batch Normalization

：將error surface landscape 剷平的方式之一

ex. w1, w2斜率差異大，不同learning rate難train

![image.png](attachment:71c31cda-308e-4bff-9b3a-18613a7da3f1:image.png)

ans:

將w1, w2 Normalization 成相同range

Feature = $(x^1, \cdots, x^R), x^i$ is feature vector

**TRAINING**

for each dimension i:

$x^i$ mean: $m^i$

$x^i$ std: $\sigma^i$

$x^r_i \leftarrow \frac{x^r_i-m_i}{\sigma_i}$

then $\forall x^i$ ~ $N(0, 1)$

but, z through w1 not nomralized

beaause $Var(z_i)=Var(\sum W_{ij}x_{j})=\sum W_{ij}^2Var(x_j)$

$Var(z_1)≠Var(z_2) \cdots$

![image.png](attachment:273c2b48-d17a-43cf-a6d3-b158f24bb2f0:image.png)

so also need to normalization z or a( if used sigmoid can normaliza z, otherwise a)

![image.png](attachment:f729359b-8bb9-4b6f-b52a-84fbd1c74ba9:image.png)

other problem: mean and std depend on $\sum z_i$, memory size limit

so used batch size get mean and std to → polulation mean, std

**TESTING**

- problem: testing data not normalized
    
- action:moving average(momentum of mean, std)
    
    - $\bar{\mu} \leftarrow p \bar{\mu}+(1-p)\mu_{batch} \\ \bar{\sigma}^2 \leftarrow p \bar{\sigma}^2+(1-p)\sigma^2_{batch}$
    
    ![image.png](attachment:16d0422f-faa5-4989-8d6a-5978e6f93f33:image.png)
    

**Problem: Internal Covariate Shift**

每層的輸入一直在更新，後面幾層(weight:B)更新為前一狀態的輸入梯度(a)，並不是當前狀態(a’)的

![image.png](attachment:8a6351e5-c6eb-468a-8182-69de88198511:image.png)

Sol: **Batch Normalization: let a, a’ distribution more similar**

$\bar{a} = \frac{a-\mu_{batch}}{\sigma_{batch}}$ and $\bar{a'} = \frac{a'-\mu_{batch}}{\sigma_{batch}}$ ~ $N(0,1)$

### CNN

![[Pasted image 20260731104049.png]]

target $y :\text{one hot vector}= (\dots , 0, 1, 0, \dots)\in R^{\text{num of target classes}}$

![[Pasted image 20260731104109.png]]

image: 3-D tensor $H*W*C$, C: channels = 3, RGB, every element represent color intensity of color pixel

flatten( 3-D tensor ) → $R^{H*W*C}$, 因為neural network只輸入向量

if 1000 neural , 需要 $3*10^7$ weights

![[Pasted image 20260731104210.png|469]]

**neural based explaination**

image 本身特性不需要fully connection( neural → all xi)

- Neural just need identify some critical patterns
    
    - Neural just scan specific position receptive field(can be overlapped)
        
        ![[Pasted image 20260731104544.png]]
        
        
- Same patterns appear in different regions
    

Parameter sharing

different input used same weight

![[Pasted image 20260731104604.png]]


**Filter based explainatation**

model - conv → feature map

- filter parameter: by model training(gradient descent)
    
- feature map size =
    
    H, W : $\frac{k+2p-1}{s}+1$
    
    C: number of filters from convolution layer, $F \in R^C$
    

Receptive field (Neural based)== Filter(Filter) function in same channel
$RF \in R^{H*W*C_{\text{prev layer output channel}}}, Kernel \in R^{H*W*C_{\text{Orig input channel}}(\text{ex.RGB})}$  

![[Pasted image 20260731104716.png]]

|                   | Neural based                                              | Filter based                                      |
| ----------------- | --------------------------------------------------------- | ------------------------------------------------- |
| pattern identify  | Each neuron only consider a receptive field               | There are a set of filter detecting small pattern |
| Scan entire image | Neuron with different receptive field share the parameter | Each filter convolves over the input image        |

**pooling**: retain significance feature , reduce the image size

Flatten → FC layer → softmax

### Application: alpha go

vector: 19*19, black:1, white:-1, none:0, channel:48

why suitable for go playing

ans:

- used small pattern to judge → conv pattern identify
- pattern can appear in any regions → parameter sharing