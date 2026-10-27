
![[Pasted image 20261009145538.png]]

```algo
reverse(S: Single Link List)
{
	Node p = nil; // prev node
	Node q = nil; // curr node
	Node r = S.next; // next node
	
	while (r != nil){
		p = q;
		q = r;
		r = r.next;
		q.next = p;
	}
	S = q;
}
```