## Kruskal's Algorithm [Undirected Only]

(prerequisite: [Disjoint Set Union](/dsu.md))

Kruskal only thinks about edges, and its time complexity is directly tied to edges
As DSU operations (union/find) are almost constant

### The Idea
We sort all the edges, start from smallest. Everytime we pick a edge, we check are they already
connected via DSU, if not connect them. If they already connected, we can discard. They connected before means
it was done with better edges or another path

Kruskal also handles multiple edges between two nodes without any issue, as we take the best one and discard others

Example Solution: https://leetcode.com/problems/min-cost-to-connect-all-points/description/
```python

class Solution:
    
    def minCostConnectPoints(self, points: List[List[int]]) -> int:
        n = len(points)
        rank = [1] * n
        parent = [x for x in range(n)]
        def union(n1, n2) -> bool:
            # Implemented in DSU along with dependency find function
            pass
        edges = []
        for i in range(n):
            for j in range(i+1, n):
                cost = abs(points[i][0] - points[j][0]) + abs(points[i][1] - points[j][1])
                edges.append([cost, i, j])
        '''
        Actual Kruskal starts here
        Sort the edges based on cost and
        Keep union until we have exactly n-1 vertex in union
        '''
        edges.sort()
        total_cost = 0
        edges_count = 0
        for cost, x, y in edges:
            if union(x, y): # Came under same tree
                total_cost += cost
                edges_count += 1
                # Early termination
                # All MST with n nodes has exactly n-1 edges, that's wy it's a tree
                if edges_count == n - 1:
                    break 
        return total_cost
```

Time Complexity: ElogE for sort, and it's the dominating factor
Space Complexity: O(E)