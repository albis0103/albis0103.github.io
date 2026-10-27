GoogLeNet(Inception v1): google develop CNN structure in 2014, Core contribution: **Inception module**

**Inception module**
![[Pasted image 20260731110903.png]]
: Different feature need **different receptive field**, Inception module **run multiple Transformation** in single layer.
- $1 \times 1$ Convolution
- $3 \times 3$ Convolution
- $5 \times 5$ Convolution
- $3 \times 3$ max pooling

note: $1 \times 1$ convolution
	used to Reduce dimension of Channel
	![[Pasted image 20260731110946.png]]

