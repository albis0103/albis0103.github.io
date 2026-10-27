# [#58] Jump Game

- DS or Algo: Greedy
- Date:0724
- Status: AC

## What went wrong
DFS will TLE, used greedy
(skip if solved cleanly)

## Problem
given array nums, return True if can reach last index
ex.
**Input:** nums = [2,3,1,1,4]
**Output:** true
**Explanation:** Jump 1 step from index 0 to 1, then 3 steps to the last index.

**Input:** nums = [3,2,1,0,4]
**Output:** false
**Explanation:** You will always arrive at index 3 no matter what. Its maximum jump length is 0, which makes it impossible to reach the last index.

## Code
$\text{canJump(nums)}$
	$\text{n}\leftarrow 0$ 
	$LastPos \leftarrow n-1$ 
	$\text{for i = 0 to n-1 do}$ 
		$\text{if maxReach == n-1 return True}$
		$\text{else if i > maxReach return False}$
		$\text{maxReach = max(nums[i]+i, maxReach) //Greedy Strategy}$
	$\text{returm False}$		

```python
def canJump(nums:List[int]) -> bool:
	n = len(nums)
	LastPos = n-1
	for i in range(n):
		if MaxReach >= LastPos:
			return True
		elif i > MaxReach:
			return False
		MaxReach = max(nums[i]+i, MaxReach)
	
	
```
## Redo?