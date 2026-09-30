
case for : $l ≤ r$
```Algo
int left = 0, right = n - 1;

while (left <= right){
	int mid = (left + right) / 2;
	if (target == nums[mid]) return mid;
	...
}
return -1; // non existed
```
- Finish State : $l = r_1\;\text{or}\;r = l-1$,  區間不存在
- Used for: 目標可能不存在


case for : $l < r$
```Algo
int left = 0, right = n - 1;

while (left < right){
	int mid = (left + right) / 2;
	...
}
```
- Finish state : $l=r=mid$  ,剩一個答案
- Used for : 答案一定存在

### Note : Interpolation Search

Extension of binarySearch, used `pos` to replace `mid` 
$$\text{pos} = \text{low} + \frac{\text{target} - \text{nums[low]}}{\text{nums[high]}-\text{nums[low]}}\cdot(\text{high} - \text{low})$$
