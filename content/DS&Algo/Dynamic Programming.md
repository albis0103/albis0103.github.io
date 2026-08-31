
**Knapsack Problem**
: n items( $i^{th}$ item weight $w_i$ , value $v_i$ ), knapsack can carrying $K$  weight

- Fractional KP: can take Fractional item
	- **Greedy strategy**: Take Maximum Unit Value $\frac{v_i}{w_i}$ of Item
- 0/1 Kapsack Problem: Item have been Taken or non-Taken
- **Dynamic Programming**: consider item $1,..,i$ , knapsack $k$ capacity, maximum profit $c(i, k)$ 
	- $i=1, .., n\;k=1, .., K$ 
	- $c(i, k)=\begin{cases}0, &\text{if }i=0\;or\;k=0\\ c(i-1, k), &\text{if }w_i > k \\ max(c(i-1, k), v_i+c(i-1, k-w_i))&\text{if }w_i≤k\end{cases}$ 


**Longest Common Subsequence**
- $\mathbf{X}=<a,b,c,a>,\; \mathbf{Y}=<a,c,b,c>,\; LCS=<a,b,c>$ 
- **Dynamic Programming** : $LCS(X_i, Y_j)$ length $c(i,j)$ , point LCS char arrow $a(i, j)$
	- $i=1,..,m\;j=1,..,n$ 
	- $c(i, j)=\begin{cases}0, &\text{if }i=0\;j=0\\1+c(i-1,j-1), &\text{if }x_i=y_j\\ max(c(i-1, j), c(i, j-1)), &\text{if }i≠j\end{cases}$ 
	- $a(i, j)=\begin{cases}\nwarrow, & \text{if } x_i=y_j \quad(\text{From }c(i-1,j-1))\\\uparrow\, & \text{if } x_i\neq y_j \text{ and } c(i-1,j)\ge c(i,j-1)\\\leftarrow, & \text{if } x_i\neq y_j \text{ and } c(i-1,j)< c(i,j-1)\end{cases}$  

- Application: **Longest Increasing Subsequence**
	- $\mathbf{X}=<5,1,3,2,4>, \; LIS=<1,2,4>$ 
	- sol:
		1. $\mathbf{Y}=sort(\mathbf{X})$ 
		2. $\text{return }LCS(\mathbf{Y}, \mathbf{X})$ 

**Minimum Edit Distance**
- Two String $A=a_1a_2\cdots a_n, B=b_1b_2\cdots b_n$ can operate the char(Insert, Delete, Replace)
- return minimum number of operation
- The minimum number of operating $A_i, B_j$ : $c(i, j)$
	- $c(i, j)=min\begin{cases}c(i-1, j-1), &\text{if }a_i = b_i \\ c(i-1, j)+1, &\text{if }a_i ≠ b_i,  \text{delete }a_i \\ c(i, j-1)+1&\text{if }a_i ≠ b_i \text{insert }b_j\\ c(i-1, j-1)+1, &\text{if }a_i ≠ b_i\;a_i\text{ replace to} b_j\end{cases}$ 