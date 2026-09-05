## Graph Representation - Adjacency Matrix

For floyd-warshall algorithm, the standard practice is adjacency matrix, and we need to know 
the no of vertices before head, and construct a 2D matrix

```python
import math

# ---------------------------------------------------------
# SCENARIO A: Weighted Graph (Shortest Path Cost)
# ---------------------------------------------------------
adj_matrix = [[math.inf] * n for _ in range(n)]

# FIX 1: Set distance to self as 0
for i in range(n):
    adj_matrix[i][i] = 0

for x, y, cost in edges:
    adj_matrix[x][y] = cost
    # adj_matrix[y][x] = cost  # Uncomment for undirected


# ---------------------------------------------------------
# SCENARIO B: Unweighted Graph (Shortest Path / Minimum Hops)
# ---------------------------------------------------------
adj_matrix = [[math.inf] * n for _ in range(n)]

for i in range(n):
    adj_matrix[i][i] = 0

for x, y in edges:
    # FIX 2: Set edge weight to 1 instead of True
    adj_matrix[x][y] = 1 
    # adj_matrix[y][x] = 1  # Uncomment for undirected


# ---------------------------------------------------------
# SCENARIO C: Unweighted Graph (Reachability ONLY - "Warshall's Algorithm")
# ---------------------------------------------------------
adj_matrix = [[False] * n for _ in range(n)]

for i in range(n):
    adj_matrix[i][i] = True  # A node can always reach itself

for x, y in edges:
    adj_matrix[x][y] = True
    # adj_matrix[y][x] = True  # Uncomment for undirected
```

# Floyd Warshall Algorithm
All pair shortest path. It's like running dijkstra for every single source. This is one of the easiest and most versatile graph algorithm, works on any kind of graphs and
finds all pair shorted path distance / reachability. The output is n*n matrix where we can have 

Works on directed or undirected, weighted or unweighted, and handles negative edge weights, but not negative cycles, which it instead detects via matrix[i][i] < 0
only three for loops
```python
for k in range(n):
    for i in range(n):
        for j in range(n):
            matrix[i][j] = min(matrix[i][j], matrix[i][k] + matrix[k][j])
```

for reachability detection using boolean matrix
```python
matrix = [[False] * n for _ in range(n)]

for i in range(n):
    matrix[i][i] = True
for x, y in edges:
            matrix[x][y] = True # directed + unweighted
            # matrix[y][x] = True undirected unweighted graph

        for k in range(n):
            for i in range(n):
                for j in range(n):
                    matrix[i][j] = matrix[i][j] or (matrix[i][k] and matrix[k][j])
```