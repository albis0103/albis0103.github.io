# [#236]  Lowest Common Ancestor of a Binary Tree

- DS or Algo: Postorder
- Date:0730
- Status: AC 

## What went wrong

confusion with return value

## Problem
given root, p, q, return p and q lowest common ancestor![[Pasted image 20260731093231.png]]
## Code
$\text{if root = p or q return root;}$
$\text{if !root return None;}$ 
$\text{postorder recursion: find leftLCA, rightLCA}$ 
$return \begin{cases}root, & if\; \exists \; \text{Both point(leftLCA and rightLCA)} \\ \text{leftLCA or rightLCA}, &if \; \exists \; \text{Single point(leftLCA or rightLCA)} \end{cases}$ 
```python
def lowestCommonAncestor(self, root: 'TreeNode', p: 'TreeNode', q: 'TreeNode') -> 'TreeNode':
	if not root:
		return None
	if root == left or root == right:
		return root
		
	leftLCA = self.lowestCommonAncestor(root.left, p, q)
	rightLCA = self.lowestCommonAncestor(root.right, p, q)
	
	if leftLCA and rightLCA:
		return root
	return leftLCA or rightLCA
```

```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q){
	if(root == null)return root;
}
```

note: if BST
	$\text{Top to Down;}$
	$root = \begin{cases}root\rightarrow right , & if \; \text{root}<p\text{ and }q \\ root\rightarrow left , & if \; \text{root}>p\text{ and }q\\root,& if \; p<\text{root}<root\end{cases}$  
## Redo? yes