
**Motivation**

|             | Update | Find        |
| ----------- | ------ | ----------- |
| Linked List | $O(1)$ | $O(n)$      |
| Array       | $O(n)$ | $O(log\;n)$ |
**Skip List**
- is Dynamic Data Structure similar to Linked List, **Update**$O(1)$ 
- also can Binary Search similar to Array, **Find** $O(log\;n)$ 
- Randomized Data Structure, each node Height $H_i \sim Geo(p)$ [[Random Variable]]

### Find
![[Pasted image 20260928231056.png]]


### Construction
1. Sorted Linked List $\rightarrow$ Skip List
```algo
for each node
	list.addLast(node)
	do{
		randomized decide node growing high(p = 1/2)
	}until(node.height >= list.height)
	
```
![[Pasted image 20260928231757.png]]
![[Pasted image 20260928231810.png]]
2. Size
	- The node at layer $k$  
	- let $i = j - k$ 
$$P(L≥k)=\sum_{j=k}^{\infty}p^j(1-p) = (1-p)\;p^k\sum_{m=0}^{\infty}p^m = (1-p)\;p^k \cdot \frac{1}{1-p}=p^k$$
$$E[\text{totalNode}]=\sum_{i=1}^n\sum_{k=0}^{\infty}p^k = \frac{1}{1-p^k}\cdot n = O(n)$$

**Perfect Skip Lists**
each Layer, layer $i$ nodes $n_i = \frac{1}{2}n_{i+1}$  