# Khan's Algorithm [Directed Only]


We can use this to do three different things

- Topo Sort
- Cycle Detection
### The Concept of indegree

Indegree is the number of edges directed into a vertex in a directed graph. The below image
is self-explanatory

![indegree](/assets/images/indegree.png)

### How Khans Algo works ?
Initially create an indegree dict/list, where indegree[x] == #incomming_edges


We keep s que, where initially we insert all the zero indegree items.
Then as we pop items, we reduce all childs indegree by one, as that parent is done now. We
Treat indegree as dependency, reducing them one by one until its 0 and we can also mark it as done

## Topological Sort using Khans Algorithm

```python
from collections import defaultdict, deque

def topological_sort_known_n(n, edges):
    adj_list = defaultdict(list)
    indegree = [0] * n 

    # Build the adjacency list and calculate in-degrees
    for x, y in edges:
        
        adj_list[x].append(y)  # Dependency: x -> y (x must come before y)
        indegree[y] += 1
        # For some problems, like course schedule, the input is inverted
        # in those scenarios, we need to swap the variable places
        '''
        for x, y in prerequisites:
            adj_list[y].append(x)
            indegree[x] += 1
        '''
        
    # Initialize the queue with all nodes having 0 in-degree
    que = deque(i for i in range(n) if indegree[i] == 0)
    order = []

    # Process the graph
    while que:
        node = que.popleft()
        order.append(node)
        
        for child in adj_list[node]:
            indegree[child] -= 1
            if indegree[child] == 0:
                que.append(child)
                
    return order
```

Another way, what if we don't know numbers of distinct nodes, and we need to find them out from edges

```python
from collections import defaultdict, deque

def topological_sort_dynamic(edges):
    adj_list = defaultdict(list)
    indegree = defaultdict(int)
    
    for x, y in edges: # Assuming the edge is x->y, can be reverse for example course schedule
        adj_list[x].append(y)
        indegree[y] += 1
        
        # Accessing indegree[y] forces defaultdict to create a default entry (0)
        # ensuring independent source nodes are captured.
        # If Y is already there with some values, this does nothing, just a safeguard
        indegree[y]
    
    # Initialize the queue with nodes that have an in-degree of 0
    que = deque(node for node, ind_count in indegree.items() if ind_count == 0) 
    order = []
    
    while que:
        node = que.popleft()
        order.append(node)
        
        for child in adj_list[node]:
            indegree[child] -= 1
            if indegree[child] == 0:
                que.append(child)
                
    return order
```

## Cycle Detection using Khans Algo
Kahn’s algorithm can inherently detect deadlocks. If a graph contains a directed cycle, 
the nodes involved in that loop will never see their indegree drop to $0$. They will remain trapped inside the 
dependency bubble and never enter the queue.Therefore, if the number of elements processed in our topological 
sort is less than the total number of unique elements in our graph, a cycle must exist.

```python
from collections import defaultdict, deque

def has_cycle_bfs_static(n, edges):
    adj_list = defaultdict(list)
    indegree = [0] * n  # Fixed array

    for x, y in edges: # Assuming x -> y
        adj_list[x].append(y)
        indegree[y] += 1

    que = deque(i for i in range(n) if indegree[i] == 0)
    visited_count = 0

    while que:
        node = que.popleft()
        visited_count += 1
        for child in adj_list[node]:
            indegree[child] -= 1
            if indegree[child] == 0:
                que.append(child)

    return visited_count != n
```


Dynamic when we don't know nodes count beforehand and there is a chance of totally isolated nodes
```python
from collections import defaultdict, deque

def has_cycle_bfs_dynamic(edges):
    adj_list = defaultdict(list)
    indegree = defaultdict(int)

    for x, y in edges: # Assuming x -> y
        adj_list[x].append(y)
        indegree[y] += 1
        indegree[x]  # Force 'x' into the dictionary with default 0

    que = deque(node for node, count in indegree.items() if count == 0)
    visited_count = 0

    while que:
        node = que.popleft()
        visited_count += 1
        for child in adj_list[node]:
            indegree[child] -= 1
            if indegree[child] == 0:
                que.append(child)

    # Compare against the dynamic length of our dictionary
    return visited_count != len(indegree)
```
