
**DFS**
(Recursion)
```Algo
visited[1..n] = False
DFS(s: vertex)
{
	visited[s] = True;
	for each v : G.adj[s] do
		if(!visited[v]) DFS(G, v)
}
```
(iterative: stack)
```Algo
DFS(s: vertex)
{
	vector<bool> visited[n];
	stack<vertex> stack;
	stack.push(s);
	while(!stack.Isempty)
	{
		s = stack.pop();
		if !visited[s]
			visited[s] = True;
			for each v : G.adj[s] do
				if !visited[v]
					stack.push(v)
		
	}
}
```


**BFS**
```Algo
visited[1..n] = False;
BFS(s: vertex)
{
	Queue<vertex> Q;
	Q.enqueue(s);
	visited[s] = True;
	while(!queue.Isempty)
	{
		u = Q.dequeue()
		for each v: G.adj[u] do
			if !visited[v]
				visited[v] = True;
				Q.enqueue(v);
				
	}
}
```