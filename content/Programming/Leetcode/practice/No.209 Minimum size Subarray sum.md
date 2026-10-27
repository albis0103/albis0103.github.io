# [#209] Minimum size Subarray sum

- DS or Algo: Two-Pointer, Slinding-Window, Array
- Date:0422
- Status:AC 

## What went wrong


(skip if solved cleanly)

## Problem
given array nums, and int target , retrun minimum number of items ,
let sum of items from array == target

## Code
```python
class Solution:
	def minSubArrayLen(self, target: int, nums: List[int]) -> int:
	n = len(nums)
	cur_sum, win_len = 0, n+1
	left = 0
	for right in range(n):
		cur_sum += nums[right]
		while (cur_sum >= target):
			if right-left+1 < win_len:
				win_len = right-left+1
			ur_sum -= nums[left]
			left += 1

	return win_len if win_len <= n else 0

'''
[2,3,1,2,4,3]
L
R
'''
```

## Redo?