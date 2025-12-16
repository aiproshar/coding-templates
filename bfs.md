
## Defining Node Class

```python
class GraphNode:
    def __init__(self, property):
        '''
        Value can be anything, any property that is linked with the node,
        
        Point to be noted: For a graph, the weight is a property of edge, not node.
        '''
        self.property = property
        self.neighbors = list(GraphNode)
        # For weighted   graph
        self.neighbours = list((GraphNode, weight:int))
```

## Using Adjacency List

```python
#Represent Adj List using defaultdict

adj_list = defaultdict(list)

# x -> y : a node in undirected graph

adj_list[x].append(y)
adj_list[y].append(x)

# For directed unweighted graph
adj_list[x].append(y)

# For directed weighted graph
adj_list[x].append((y, weight))

```

### Basic BFS


```python
def bfs(source, adj_list: defaultdict(list)) -> None:
    
    # BFS needs a queue
    que = deque()
    que.append(source)
    
    # Visited set, this can be also an list
    visited = {source}

    while que:
        parent = que.popleft()
        for child in adj_list[parent]:
            # Logic
            # Logic
            if child not in visited:
                # The child is not explored, lets put him in queue
                que.append(child)
                #also mark him as explored
                visited.add(child)
```

### BFS with level order traversal

```python
def bfs(source, adj_list: defaultdict(list)) -> None:
    
    # BFS needs a queue
    que = deque()
    que.append(source)
    
    # level exploration, we populate as we traverse

    level = 0
    depth = {source: 0}

    while que:
        for _ in range(len(que)):
            parent = que.popleft()
            #logic
            #logic
            for child in adj_list[parent]:
                if child not in depth:
                    #logic
                    que.append(child)
                    depth[child] = level + 1 # Add 1 to parent level
        level += 1 
```




## BFS in 2D grid, most important types

```python
    def __bfs(self, i, j, grid: list[list[int]]) -> None:
        queue = deque()
        queue.append((i, j)) #-> Queue of co-ordinates
        grid[i][j] = "0"
        # The direction is the most important thing, it represents the childs
        # Also need to do bound checking

        #left right updown
        direction = [(1, 0), (-1, 0), (0, -1), (0, 1)]
        # For knight moves
        direction = [   (2, 1), (2, -1), (-2, 1), (-2, -1),
                        (1, 2), (1, -2), (-1, 2), (-1, -2)
        ]
        # updown leftright with corners
        directions = [
            (-1, 0),   # up
            (1, 0),    # down
            (0, -1),   # left
            (0, 1),    # right
            (-1, -1),  # up-left
            (-1, 1),   # up-right
            (1, -1),   # down-left
            (1, 1)     # down-right
        ]

        while queue:
            x, y = queue.popleft()
            # This convention is important for clean code
            # dx, dy -> delta
            # nx, ny -> new coordinates
            for dx,dy in direction:
                nx, ny = x + dx, y + dy
                if 0 <= nx < len(grid) and 0 <= ny < len(grid[0]) and grid[nx][ny] == "1":
                    grid[nx][ny] = "0"
                    queue.append((nx, ny))
                    
```

