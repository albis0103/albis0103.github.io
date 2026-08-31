# [#70] Climbing Stairs

- DS or Algo: DP, Recursion, Fibonacci
- Date:07/01
- Status: AC

## What went wrong
Get confused of 
Number of stair:$1+(n-1)+2+(n-2)$
and 
Number of method: $(n-1)+(n-2)$

(skip if solved cleanly)

## Problem
$n$ step to reach the top, each time can only 1 step or 2 step
## Code
$$c(n)=\begin{cases}1,&\text{if 1 step} \\ 2,&\text{if 2 step}\\c(n-1)+c(n-2),&\text{if n steps}\end{cases}$$ 
```python
def climbStairs(self, n: int) -> int:
	dp = [0] * (n+1)
	if n >= 1:
		dp[1] = 1
	if n >= 2:
		dp[2] = 2
	for i in range(3,n+1):
	  dp[i] = dp[i-1] + dp[i-2]
	return dp[n]
```
## Redo?