Build - Heap(Bottom up, MaxHeap)
- `Adjust(tree, i, n)`
```Algo
Adjust(tree, i, n)
{
	j = 2*i;
	k = tree[i];
	while(j <= n)
	{
		if j < n
			if tree[j] < tree[j+1]
				j++
			if k < tree[j] break;
			else
				tree[j/2] = tree[j]
				j *= 2
	}
	tree[j/2] = k
}
```
- `createHeap(tree, n)`