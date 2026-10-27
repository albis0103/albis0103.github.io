將特徵子及選擇問題（組合最佳化問題）轉為GA(Genetic Algorithm)搜尋空間用演化式搜尋(Evolutionary Search)找近似最優解

#### Problem:
Given Feature Set $\mathbf{X}=\left\{x_1,\cdots,x_n\right\}$, find Subset $\mathbf{S} \subset \mathbf{X}$ let fitness function be maximum $\max f(S)$

- note: fitness function $f$
$$f(S)=\text{Accuracy}(S)-\lambda\cdot \frac{|S|}{n}$$

#### 染色體 （Chromosome Representation）
染色體 $c \in \mathbb{R}^n$  有 $n$ 個基因(特徵)
$$c = (b_1,\cdots,b_n),\;b\in\{0,1\}$$

#### Evaluation
https://dotblogs.com.tw/dragon229/2013/01/03/86692

![[Pasted image 20261009114648.png]]
- Initialization: 隨機產生 $m$ 個 染色體(bitmask$=c_1,\cdots, c_m$)
	- $m$: population size 族群數（hyperparameter）
- Evaluation: 計算 $\{\;f(c_i)\;\}_{i=1}^m$ 
- Selection: 選父代：留下好的，排除不好的。依 $f(c_i)$ 選擇
	- Roulette wheel selection: 算全部，選中機率 $\frac{f(c_i)}{\sum_j f(c_j)}$ 
	- Tournament selection: 隨機抽Top - k選最好
- Cross-over: 隨機配對兩兩一組交配，以$P_c$ 機率決定學定 基因 $b_j$ 是否交換
	- 交配位置：One - point, Uniform
![[Pasted image 20261009114814.png]]
- Mutation: 以 $P_m$ 決定是否隨機突變(bit隨機翻轉：$0\rightarrow 1, \; 1 \rightarrow 0$)，避免過早收斂
![[Pasted image 20261009114823.png]]
- Replacement：子代取代父代（全部 or 部分）