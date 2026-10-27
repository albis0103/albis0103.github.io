![[20152236aLZnLVzqGm.jpg]]

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


**Unet**
![[Pasted image 20260731103715.png]]

- **Contracting Path/ Encoder**: Two $3*3\text{Conv}$ + $\text{ReLU}$ 
	: each layer Down sampling will let Half image resolution, Double Channel($64\rightarrow 128\rightarrow 256\rightarrow 512 \rightarrow 1024$)

- **Bottleneck**: same convolution, give global context to decoder not skip 

- **Expansive Path/ Decoder**: Two $2*2\text{up-Conv}+\text{channel-wise concatenation}$
	:each layer Up sampling will let Double image resolution, Half Channel
	$\text{channel-wise concatenation}:$ 
		$x^i_{decoder}=Concat(Upsample(x^{i-1}_{decoder}), x^i_{encoder})$ 

- **Skip - Connection**
	:transport encoder output to correspond decoder layer, Restore spatial detail information loss on downsampling



- Standard CNN problem
	:On Inpainting Tesk(image restoration or replacement) will identically treat other invalid pixel(0 or Nan .. etc value), cause color discrepancy and blurriness


**PConv(Partial Convolution)**
:only output valid value

$x' = \begin{cases}W^T(X \odot M)\frac{sum(\mathbf{1})}{sum{M}}+b,& \text{if sum(M)>0}\\0,& \text{otherwise}\end{cases}$  

note
	$\frac{sum(\mathbf{1})}{sum{M}}$ : used to renormalization, because when$\text{num of pixel} \downarrow$ (masked)will let $\text{size} \downarrow$ 
mask - update: mask will update before each partial convolution
	$m'=\begin{cases}1,& if sum(M)>0 \\ 0, & if sum(M)=0\end{cases}$ 
	so if exist pixel, next layer will be valid

**GConv(Gated Convolution)**
: let model learn **Dynamic mask** $\in [0,1]$ 
	$y = \phi(W_f * X) \odot \sigma(W_g * X)$
		$\phi$:non-linear activation function(common way : LeakyReLU [[ReLU]], output: tanh)
		$W_f$: generate feature
		$W_g$: generate gating signal

[[ConvLSTM(Convolution LSTM)]]
[[GoogLeNet]]