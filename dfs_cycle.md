## DFS 

We can use DFS for cycle detection in directed graph. In undirected graph, cycle detection means the graph is a not valid tree
Using DFS we can also find finish time, which's reverse is topological sort

### So using DFS
- **Directed Graph**: Cycle Detection, DAG or DCG ? Topo sort
- **Undirected Graph**: Valid Tree or Not ? Forest ? How many trees in the forest ?


### Cycle Detection in Directed Graph

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
        if dfs(source, adj_list, color) is True:
            print("We have backedge and cycle")
print("No cycle detected in graph")

```


### Topo Sort in a directed graph → Almost same as cycle detection, we just push Nodes as they become BLACK

Because a node only turns Black after all of its downstream dependencies are completely processed, recording the nodes in the reverse order of them turning Black gives us a perfectly valid topological sort.



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