# [#11] Container With Most Water

- DS or Algo: two pointer, silding window , greedy($h[i] > h[j], j-1$)
- Date:0822
- Status: AC 

## What went wrong
The judge with i or j select to be forward is made Confusion, used Greedy strategy: 
$$\begin{cases}j--, &\text{if }height[i]>height[j] \\ i++,  &\text{if }height[j]>height[i]\end{cases}$$
(skip if solved cleanly)

## Problem

## Code
```python
class Solution:

def maxArea(self, height: List[int]) -> int:
	i, j = 0, len(height)-1
	MaxArea = 0
	while (i < j):
		h = min(height[i], height[j])
		w = j-i
		MaxArea = max(MaxArea, h*w)
		if height[i] > height[j]:
			j--
		else:
			i++
	return MaxArea

'''

height =

[1,8,6,2,5,4,8,3,7]

i j, w = 8, area = 8, h[i] < h[j], i++

i j, w = 7, area = 49, h[j] < h[i], j--

i j , w=6, area = 18, h[j] < h[i], j--

...

  

'''
```

## Redo?