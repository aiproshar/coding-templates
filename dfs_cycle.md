## DFS [Directed, Undirected]

We can use DFS for cycle detection in directed graph. In undirected graph, cycle detection means the graph is not a valid tree
Using DFS we can also find finish time, whose reverse is topological sort

### So using DFS
- **Directed Graph**: Cycle Detection, DAG or DCG ? Topo sort
- **Undirected Graph**: Valid Tree or Not ? Forest ? How many trees in the forest ?


# Cycle Detection in Directed Graph

The core logic is simple, we try to do back-edge detection. What is a back-edge ?
A back-edge is we going back to the same node which is in current recursion stack. A cycle is formed

```python
def dfs(self, source, adj_list: defaultdict(list), color: list) -> bool:
    # Return true if we detect a backedge means cycle
    # Return False if no cycle is detected during graph traversal
    WHITE, GREY, BLACK = 0, 1, 2

    # Grey means its in current recursion stack
    color[source] = GREY
    for child in adj_list[source]:
        if color[child] == GREY:
            return True
        if color[child] == WHITE and dfs(child, adj_list, color):
            return True
    # Its fully visited including its child. We are taking out of the STACK, MARKING AS BLACK
    # SO WE DO NOT visit this again
    color[source] = BLACK
    return False
```
### Corner Case
What if our source is not SCC ? we cannot reach to parts of the graph where there is a cycle ?
That's why we need to mark visited/explored nodes as black, and iterate through all the nodes

```python
# Iterate over every node
for source in sources:
    if color[source] == WHITE:
        # We are trying to detect a cycle
        if dfs(source, adj_list, color):
            # Return as soon as we find a cycle, don't keep going
            return False
# Every source explored, no back-edge found
return True
```


### Topo Sort in a directed graph → Almost same as cycle detection, we just push Nodes as they become BLACK

Because a node only turns Black after all of its downstream dependencies are completely processed, recording the nodes in the reverse order of them turning Black gives us a perfectly valid topological sort.

This only holds if the graph is a DAG. If there is a cycle there is no valid topological order at all, so run the same back-edge check first and bail out on a cycle.



```python
def dfs(self, source, adj_list: defaultdict(list), color: list, finish_time) -> bool:
    # Return true if we detect a backedge means cycle
    # Return False if no cycle is detected during graph traversal
    WHITE, GREY, BLACK = 0, 1, 2

    # Grey means its in current recursion stack
    color[source] = GREY
    for child in adj_list[source]:
        if color[child] == GREY:
            return True
        if color[child] == WHITE and dfs(child, adj_list, color, finish_time) :
            return True
    # Its fully visited including its child. We are taking out of the STACK, MARKING AS BLACK
    # SO WE DO NOT visit this again
    color[source] = BLACK
    finish_time.append(source)
    return False


# finish_time[::-1] -> the topological sort, reverse of finish time
```
Note: same as before, we must run DFS on each source, if source already BLACK skip. This includes totally disconnected and non SCC nodes

# Cycle detection in undirected graph [HARD]

In an undirected graph, a connected graph that isn't a Tree has a cycle (a disconnected acyclic graph is a forest, not a cycle).
One quick way is a tree of n nodes will always have n-1 edges, not more not less.
But even with n-1, there can be forest, two completely separated trees, so we also need to check that

Early Termination: A undirected graph without cycle is tree, so a tree with n nodes must have n-1 edges, but we also need to check for forest and back-edge.
If we dont have exactly n-1 edges, we can early terminate

## Edge cases
- child of current node can be parent, as the graph is undirected. We must exclude this. How we do this ? We need to know the parent of 
current node, so if we send this along with the param, we good. A separate parent dict also work but overkill
- We must keep a visited set or also can use visited_count as count of nodes we mark black. If The graph is a forest, we will have less
than n vertex marked as visited. 
```python
def dfs(node, parent) -> bool:
    color[node] = GREY
    for child in adj_list[node]:
        
        #skip inserting the parent of current node
        if child == parent:
            continue
        if color[child] == GREY:
            return True
        if color[child] == WHITE:
            if dfs(child, node) is True:
                return True
    color[node] = BLACK
    visited.add(node)
    return False

if dfs(0, None) is True:
    return False
if len(visited) != n:
    return False
return True
```

### So condition to check after running DFS on undirected graph

```python
    # check for back-edge means cycle
    if dfs(0, None) is True:
        return False
    # check for forest
    if len(visited) != n:
        return False
    # No Cycle no forest, we good
    return True
```

#### Problems to solve
- https://leetcode.com/problems/graph-valid-tree/description/