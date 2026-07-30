# [#207] Course Schedule

- DS or Algo:Topology Sort[[Topology Sort]]
- Date:0722
- Status: AC 

## What went wrong
- Graph construct: used adjencent list
- topology sort terminate condition

(skip if solved cleanly)

## Problem
given numCourse: int, number of courses and prerequisites:List[[]], the prerequisites of courses, ex.$[a_i, b_i]$ mean $a_i$ is prerequisites of $b_i$ , $a_i \rightarrow b_i$ 

## Code
```python
class Solution:
def canFinish(self, numCourses: int, prerequisites: List[List[int]]) -> bool:
	graph = [[]for _ in range(numCourses)]
	indegree = [0] * numCourses
	for a, b in prerequisites:
		graph[b].append(a)
		indegree[a] += 1
	visited = 0
	Q = deque(c for c in range(numCourses)if indegree[c]==0)
	while Q:
		u = Q.popleft()
		visited += 1
		for v in graph[u]:
			indegree[v] -= 1
			if indegree[v] == 0:
			Q.append(v)
	return visited == numCourses
'''

[v, u] = u -> v , need to finish u before v

construct graph, indegree;

bfs:

	q = deque(for each course if degree[course] = 0)

	while q:

		item = q.popleft

		visited++

			for c in graph[item]:

				degree[c]--

					if degree[c] == 0 enqueue, visited++

'''
```


```java
class Solution {
    public boolean canFinish(int numCourses, int[][] prerequisites) {
        List<List<Integer>> graph = new ArrayList<>();
        for (int i = 0; i < numCourses; i++) {
            graph.add(new ArrayList<>());
        }

        int[] inDegree = new int[numCourses];
        for (int[] prerequisite : prerequisites) {
            int u = prerequisite[0];
            int v = prerequisite[1];
            graph.get(v).add(u);
            inDegree[u]++;
        }

        Deque<Integer> queue = new ArrayDeque<>();
        int visited = 0;

        for (int i = 0; i < numCourses; i++) {
            if (inDegree[i] == 0) {
                queue.offer(i);
            }
        }

        while (!queue.isEmpty()) {
            Integer u = queue.poll();
            visited++;
            for (Integer v : graph.get(u)) {
                inDegree[v]--;
                if (inDegree[v] == 0) {
                    queue.offer(v);
                }
            }
        }

        return visited == numCourses;
    }
}
```


## Redo?