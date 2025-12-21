# Disjoint Set Union / Union Find

Initially eash set is root of its own. For Any DSU operation, we need to implement two methods

    * find(node) -> find the root/leader of this union where the node belongs
    * union(node_1, node_2) -> Bring 2 nodes under one root/leader


Lets implement the function

## Required Data Structures
```python
# Initially all nodes self are its leader
parent = [x for x in range(n)] # parent[x] = x for any x


# The rank is an optional optimization, ensures smaller union joins larger one, hence less amount of leader of small groups is updated
rank = [1 for x in range(n)] # initially each union has only one dude
```

#### Initially, we have all disconnected nodes, so n disjoint sets. Whenever we are able to union two nodes, we reduce that by one

### The FIND functon

find figures out the root/leader of any node. How we do it ? Rember, a root is that node where it itself is its parent. So we continue climbing up until we find the parent

```python
# Recursive Version, without path compression
# This is for the sake of example, we dont use this
# Always use path compression

def find(x: int, parent: list[int]) -> int:
    if parent[x] != x: # means x is not a leader
        return find(parent[x], parent) #recursive call until we find a leader
    return x

# Find with path compression, same recursive call
# But during backtrack we update all the nodes parent
# This is called path compression, all the large chains now chopped
def find(x: int, parent: list[int]) -> int:
    if parent[x] != x:
        parent[x] = find(parent[x], parent)
    return parent[x]

# Hardest version, find with iterative call -> Not needed unless interviewer asks so
# No rist of stack overflow
# First find the leader/root
# Then iterate/teavese up and update the values

def find(x: int, parent: list[int]) -> int:
    root = x
    while root != parent[root]:
        root = parent[root]
    while parent[x] != root:
        x, parent[x] = parent[x], root
```

## The UNION function

The union function returns true if both nodes were in diofferent union and they are merged, or false if they are already in the same set

The union function returning false means they have detected a cycle

How we be sure ? If both nodes leader is same, then they are in the same set

```python
def union(self, node1: int, node2: int, parent: list[int], rank: list[int]) -> bool:
        # The very first operation will be finding the parents, remember, all operation must be done based on the roots. For DSU, all properties are tied with root (rank)
        root1 = self.find(node1, parent)
        root2 = self.find(node2, parent)
        if root1 == root2:
            return False
        if rank[root1] >= rank[root1]:
            # We update parent of root, path compression will update the node parents later
            parent[root2] = root1
            rank[root1] += 1
        else:
            parent[root1] = root2
            rank[root2] += 1
        return True
    # The return is very important, from that we can reduce the initial DSU count, which was equal to no of nodes. True means two set joined, hence we reduce one (you might be thinking a group of 5 nodes and 8 nodes joined, why we only decrease one ? Because when we created the 5 nodes and 8 nodes group, de decreased them already)
```


## MUST DO while revisit problems

* https://leetcode.com/problems/number-of-connected-components-in-an-undirected-graph/description/

* https://leetcode.com/problems/graph-valid-tree/description/

Note: any DSU problem can also be solved with DFS/BFS. for cycle/edge detection
