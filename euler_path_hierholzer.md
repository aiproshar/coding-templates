# Eulerian Path / Hierholzer's Algorithm [Directed]

An **Eulerian path** is a trail that uses every **edge** exactly once. An
**Eulerian circuit** is one that ends where it started.

Note the word *edge*, not *vertex*. Nodes may be revisited as often as you like —
what you must not do is reuse an edge. That is the opposite of most traversals you
already know, where a `visited` set stops you from re-entering a node.

**Hierholzer's algorithm** finds one in O(E). It is a **post-order DFS that never
backtracks** — walk edges greedily, and when you get stuck, write the node down and
return. The path builds itself **backwards**.

> **Trigger to memorize:** "use every edge / ticket / road / pair exactly once, in
> some order". If the problem hands you a multiset of edges and wants a sequence
> that consumes all of them, it is Euler, not backtracking.

You already know the shape of this. In [DFS](/dfs_cycle.md), topological sort is
"append the node when it turns BLACK, then reverse the finish times". Hierholzer's
is the same move — append the node when it has no unused edges left, then reverse.
The difference is what gets consumed: topo sort retires a **node** once and never
returns to it, Hierholzer's retires an **edge** and may come back to the node many
times.

## Does a path even exist?

Check degrees before you write any code. `out(v)` and `in(v)` are out-degree and
in-degree.

| You want | Condition |
| --- | --- |
| Circuit | `in(v) == out(v)` for every `v` |
| Path | exactly one `v` with `out(v) - in(v) == 1` (the **start**), exactly one with `in(v) - out(v) == 1` (the **end**), all others balanced |

Degrees alone are **not** enough. Every vertex that has at least one edge must
also sit in one connected piece, otherwise you get two separate valid-looking
trails and no way to join them. Vertices with no edges at all are fine — you are
covering edges, not vertices.

Finding the source is the same dynamic-dictionary trick as [Kahn's
Algorithm](/topo_sort_cycle_khans_algo.md), where nodes are discovered from the
edges rather than being `0..n-1`:

```python
def find_source(edges):
    out_deg = defaultdict(int)
    in_deg = defaultdict(int)

    for x, y in edges:              # assuming the edge is x -> y
        out_deg[x] += 1
        in_deg[y] += 1
        # Force both nodes into both dicts with a default 0, otherwise a pure
        # source (no incoming) or pure sink (no outgoing) is invisible below
        in_deg[x]
        out_deg[y]

    source = None
    for node in out_deg:
        if out_deg[node] - in_deg[node] == 1:
            source = node           # the one unbalanced node, the start
            break
    # All balanced -> it's a circuit, any node with an edge works as a start
    return source if source is not None else next(iter(out_deg))
```

Many problems hand you the source instead (Reconstruct Itinerary always starts at
`JFK`), in which case skip all of this.

## The Ah-Ha: getting stuck is not a failure

This is the part that kills the backtracking instinct.

In backtracking, hitting a dead end means *"wrong turn, undo it"*. In Hierholzer's,
hitting a dead end means *"this node is the end of the path — write it down"*.

**Claim.** If an Eulerian path from `s` to `t` exists, and you start at `s` and
greedily follow any unused edges, the **only** vertex you can ever get stuck on is
`t`. Not a wrong turn. The actual final destination.

**Proof.** Say you are sitting at `v` with no unused out-edge.

- **If `v != s`:** every earlier visit to `v` arrived once and departed once,
  except the current one, which arrived and has not departed. So
  `departures = arrivals - 1`. Stuck means every out-edge is used, so
  `out(v) = arrivals - 1`, giving `arrivals = out(v) + 1`. You cannot arrive more
  times than there are in-edges, so `in(v) >= out(v) + 1`, i.e. `in(v) > out(v)`.
  The only vertex with more in than out is `t`. **So `v == t`.**
- **If `v == s`:** the walk began here with a departure and you are back now, so
  `departures = arrivals`. Stuck means `arrivals = departures = out(s) = in(s) + 1
  > in(s)` — arriving more times than there are in-edges. Impossible. **You cannot
  get stuck at `s`** (unless `s == t`, the circuit case, where getting stuck there
  is exactly right).

So a greedy walk can never strand you somewhere arbitrary. Every dead end is
meaningful.

**What about the edges you skipped?** Remove the `s → t` trail you just walked and
every remaining vertex is balanced again (`in == out`). A balanced graph has no
dead ends, so the leftovers form **circuits** hanging off vertices already on your
trail. Hierholzer's splices each circuit in at the vertex where it attaches — and
**post-order does that splicing for free**, because a circuit discovered while
unwinding from `v` gets appended to `res` before `v` itself does.

## The Algorithm — memorize this once

```python
from collections import defaultdict, deque

def euler_path(source, adj_list: defaultdict(deque)) -> list:
    res = []

    def dfs(node):
        # Burn every edge out of this node. We never put one back -- there is
        # no undo line, no visited set, no "did we find it yet" flag
        while adj_list[node]:
            # popleft() is O(1) on a deque; list.pop(0) would be O(n)
            child = adj_list[node].popleft()
            dfs(child)
        # No edge left -> stuck -> this node ends the current trail.
        # POST-ORDER: this line goes AFTER the while, never inside it
        res.append(node)

    dfs(source)
    return res[::-1]        # built finish-to-start, flip it once at the end
```

Building the adjacency list, same convention as every other note here:

```python
adj_list = defaultdict(deque)

for x, y in edges:              # edge x -> y
    adj_list[x].append(y)
```

The one line you must not get wrong is `res.append(node)` sitting **after** the
`while`, not inside it. Move it inside and you have a pre-order DFS, which gives a
wrong path the moment any node has a dead-end branch.

Two things that are *absent* compared to a backtracking solution: there is no
`visited` set (the edge is consumed by `popleft()` and never restored), and there
is no path threaded through the recursion (`res` is filled on the way **out**, not
on the way in).

## Trace

`edges = [(0,1), (0,2), (2,0)]`, so `adj_list = {0: [1, 2], 2: [0]}`.

```text
0 -> 1                        remaining 0: [2]
  [stuck at 1] append         res = [1]
0 -> 2                        remaining 0: []
  2 -> 0                      remaining 2: []
    [stuck at 0] append       res = [1, 0]
  [stuck at 2] append         res = [1, 0, 2]
[stuck at 0] append           res = [1, 0, 2, 0]

res[::-1] -> [0, 2, 0, 1]
```

Look at what happened to node `1`. The walk picked it **first** and it was an
immediate dead end. Backtracking would undo that choice and try again.
Hierholzer's keeps it, and because it was appended first it lands **last** after
the reversal — which is exactly where it belongs. Note `0` appears twice in the
answer, which is fine: edges are used once, nodes are not.

## Variant: lexicographically smallest path

This is Reconstruct Itinerary. When several Eulerian paths exist and the problem
wants the smallest one, **sort the destinations and always take the smallest
first**. Nothing else changes.

```python
class Solution:
    def findItinerary(self, tickets: List[List[str]]) -> List[str]:
        # sort() on [x, y] pairs sorts by x first, y second. So all edges out of
        # one node land together, already in ascending order -- no per-node sort
        tickets.sort()

        adj_list = defaultdict(deque)
        for x, y in tickets:
            adj_list[x].append(y)

        res = []

        def dfs(node):
            while adj_list[node]:
                child = adj_list[node].popleft()      # smallest available
                dfs(child)
            res.append(node)

        dfs("JFK")          # the problem hands us the source
        return res[::-1]
```

Greedy is safe here for the reason the trace shows: taking the smallest edge can
never corrupt the answer, because a premature dead end is placed at the **tail**,
not the head. So the smallest choice at every step is always committable, and the
result is the lexicographically smallest path overall.

## Time and Space Complexity

- **Time: O(E)** — every edge is popped exactly once and never re-examined. It
  becomes **O(E log E)** only if the problem wants lexicographic order, and that
  `log E` is the sort, not the traversal.
- **Space: O(E)** for the adjacency lists, plus O(E) recursion depth — the trail
  can be every edge long, so watch the recursion limit on big inputs.

Compare against solving it with backtracking: that is **exponential**, roughly
`O(E^d)` for max out-degree `d`, since at each node it tries every remaining
outgoing edge, recurses fully, and unwinds on failure. It is correct, and on small
constraints it passes. But a graph with many long interlocking dead ends makes it
explore an exponential number of prefixes that Hierholzer's never looks at —
because Hierholzer's never has to ask "was that the right turn?".

## MUST DO while revisiting problems

**The canonical one**

* https://leetcode.com/problems/reconstruct-itinerary/  (LC 332, source is given as `JFK`, lexicographic tie-break)

**Euler without being told it is Euler**

* https://leetcode.com/problems/valid-arrangement-of-pairs/  (LC 2097, same algorithm — but you must find the source yourself with the `out - in == 1` rule)
* https://leetcode.com/problems/cracking-the-safe/  (LC 753, de Bruijn sequence as an Eulerian circuit; hard, and the "aha" is what the nodes and edges actually are)
