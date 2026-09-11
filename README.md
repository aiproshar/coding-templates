# Code Templates for LC

Pattern notes and reusable templates for LeetCode-style problems. Each file is
self-contained: the intuition, a template you can copy, the edge cases that
actually bite, and the problems worth revisiting.

All code is Python.

## Graphs

| Topic | What's inside |
| --- | --- |
| [BFS](bfs.md) | Adjacency list setup, basic BFS, level-order traversal with a depth map, 2D grid BFS (direction vectors, bound checks), and multi-source BFS. |
| [DFS & Cycle Detection](dfs_cycle.md) | White/grey/black colouring for back-edge detection in directed graphs, topological sort from finish times, and the undirected variant (parent skip, forest check). |
| [Kahn's Algorithm](topo_sort_cycle_khans_algo.md) | Indegree-based topological sort and cycle detection, in both known-`n` and dynamic (nodes discovered from edges) forms. |
| [Disjoint Set Union](dsu.md) | Union-Find with path compression and union by rank, why rank only bumps on ties, and the `union` return value as a cycle signal. |
| [Minimum Spanning Tree](mst_prims.md) | Prim's algorithm with a min-heap, plus the disconnected-graph corner case. |
| [Eulerian Path / Hierholzer's](euler_path_hierholzer.md) | Use every edge exactly once: the degree conditions for a path vs a circuit, the proof that a greedy walk can only get stuck at the true endpoint, post-order DFS with `deque.popleft()`, and the lexicographic variant. |
| [Floyd-Warshall](floyd_warshal.md) | All-pairs shortest path in three loops: adjacency-matrix seeding (weighted / hops / boolean), why `k` must be the outermost loop with a counterexample, negative-cycle detection, Warshall's transitive closure, and a decision guide against Dijkstra / Bellman-Ford / BFS. |

## Search

| Topic | What's inside |
| --- | --- |
| [Binary Search](bi_search.md) | One half-open skeleton for everything — left bound, right bound, exact match — plus a cheat sheet, derived queries (count, floor, ceil), Python's `bisect`, and the common pitfalls. |
| [Floyd's Cycle Detection](floyd_cycle.md) | Tortoise and hare: does a cycle exist, where does it begin (with the distance proof), and Find the Duplicate Number as the same trick in disguise. |

## Arrays & Intervals

| Topic | What's inside |
| --- | --- |
| [Intervals](intervals.md) | The overlap test, segment sweep (merge), event sweep (max concurrent), the heap alternative, interval intersection with two pointers, and the sort-by-end-time greedy. |
| [Two Pointers & Sliding Window](two_pointer_sliding_window.md) | 🚧 Stub — not written yet. |

## Sorting & Heaps

| Topic | What's inside |
| --- | --- |
| [Sorting & Heaps](sort_heap.md) | Five ways to sort without the built-in (LC 912), ordered by what to reach for under pressure: selection sort, `heapq`, merge sort, a from-scratch `Heap` class (push/pop, bubble up/down), and true in-place heapsort for the constant-space follow-up. |

## Recursion & DP

| Topic | What's inside |
| --- | --- |
| [Backtracking](backtrack.md) | The CCG framework (Choices, Constraints, Goals), the generic skeleton, and the 2D grid variant with state restore — then the two families it generates: subsets (take-it-or-leave-it, plus the sort-and-skip duplicate handling) and combinations (fixed length with early pruning, combination sum both ways). |
| [Dynamic Programming](dp.md) | The 5-step framework (objective function, base cases, recurrence, order of computation, where the answer lives), applied top-down and bottom-up to Climbing Stairs and Min Cost Climbing Stairs. |

## Bit Manipulation

| Topic | What's inside |
| --- | --- |
| [Bitmask](bitmask.md) | An int used as a **set**: the cheat sheet (test/set/clear/toggle and the precedence bug), the three low-bit idioms (`mask & -mask`, `mask & (mask - 1)`, `(1 << k) - 1`), then four patterns — one set per node (reachability, character sets), enumerating subsets and submasks, bitmask DP with the subset as the state, and state compression in BFS. Ends with why bit *tricks* are a different topic. |

## Planned

Not written yet:

- Trees (traversals, BST)
- Monotonic stack
- Prefix sums
- Dijkstra / Bellman-Ford (see [Floyd-Warshall](floyd_warshal.md) for the all-pairs case)
- Kruskal's algorithm (see [MST](mst_prims.md), currently Prim's only)
