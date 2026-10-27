Consider both Target label Relevant and dataset redundancy
Feature set: need to satisfy Max Relevance with label, Min Redundancy between feature
	- $S$: sample space
	- $c$: target class
	- $I(x, c) = H(c)-H(c|x)$ 
- Max relevance: $maxD(S, c),D = \frac{1}{|S|}\sum_{x_i \in S}I(x_i; c)$ 

- Min Redundancy:$minR(S), R=\frac{1}{|S|^2}\sum_{x_i, x_j \in S}I(x_i, x_j)$ 
**mRMR**$=D-R$ : $MI$(Mutual Information)extention

$$I(x_1, x_2; c) = I(x_1; c) + I(x_2; c \mid x_1)$$
$$I(x_1, x_2; c) = I(x_1; c) + I(x_2; c) - I(x_1; x_2) + I(x_1; x_2 \mid c)$$
$$I(x_1, x_2; c) \approx I(x_1; c) + I(x_2; c) - I(x_1; x_2)$$

$$\underbrace{I(x_1;c) + I(x_2;c)}_{\text{relevance 加總}} - \underbrace{I(x_1;x_2)}_{\text{redundancy}}$$

then $2 \rightarrow m$ 
$$max_S[\frac{1}{|S|}\sum_{x_i \in S}I(x_i; c)-\frac{1}{|S|^2}\sum_{x_i, x_j \in S}I(x_i, x_j)]$$ 