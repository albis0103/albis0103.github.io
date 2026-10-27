: find the sequence that can trace successful
	$\text{if }\exists \; e(u \rightarrow v) \text{ then u need to before v}$  
	
- **Kahn's Algorithm**: 1. compute inDegree 2. BFS enqueue, if inDegree  = 0
	$\text{contrust G: Graph<V, E>;}$
	$\text{compute: inDegree[v] }, \forall v \in G.V$ 
	$\text{Queue} \leftarrow \forall v \; \text{where inDegree[v]==0}$
	$\text{while Queue}$
		$u \leftarrow \text{Queue.dequeue}$ 
		$\text{visieted u}$
		$\text{for each v}\in \text{G.adj[u]}$
			$\text{inDegree[v]--}$
			$\text{if inDegree[v]==0} \; \text{Queue.enqueue(v)}$	
	$\text{if nums of visited ≠ Vertex number return False}$ 
	
- **DFS - based**:Run DFS, and will exists one path $u \rightarrow v$ that $u.finish > v.finish$ 