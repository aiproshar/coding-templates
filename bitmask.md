# Bitmask [Sets over a small universe]

A bitmask is an integer used as a **set**. Bit `i` is `1` if element `i` is in the
set, `0` if it is not. That is the entire idea — everything below is bookkeeping.

```
    set        binary      int
    {}         0000        0
    {0}        0001        1
    {1,2}      0110        6
    {0,1,2,3}  1111        15
```

Why bother, when Python already has `set()`?

- **Union / intersection become one CPU instruction** instead of allocating and
  copying a hash table. A set union is O(size); a bitwise OR is O(1) for practical `n`.
- **An int is hashable and immutable**, so it is a legal `@cache` key and a legal
  dict key. A `set` is neither. This alone unlocks memoized DP over subsets.
- **`range(1 << n)` enumerates every subset** of an n-element universe in one loop.

The cost: the universe must be **small, fixed, and integer-labelled**. Roughly
`n <= 20` if you are enumerating all subsets (2^20 ≈ 1e6), or `n` up to a few
hundred if you are only storing one mask per node.

> **Trigger to memorize:** you catch yourself writing `set()` where the elements
> are small integers, *or* you want to memoize on a set. Stop and use an int.

## The Cheat Sheet — memorize this once

```python
mask = 0                      # empty set
mask = (1 << n) - 1           # full set {0 .. n-1}   <- n ones

mask |= (1 << i)              # add i          (union)
mask &= ~(1 << i)             # remove i       (difference)
mask ^= (1 << i)              # toggle i
(mask >> i) & 1               # test i         -> 0 or 1
mask & (1 << i)               # test i         -> truthy / falsy

a | b                         # union
a & b                         # intersection
a & ~b                        # a minus b
a ^ b                         # symmetric difference
a & b == a                    # a is a subset of b
a & b == 0                    # a and b are disjoint

mask.bit_count()              # popcount, Python 3.10+
bin(mask).count("1")          # popcount, works everywhere
mask & -mask                  # lowest set bit, isolated (e.g. 0110 -> 0010)
mask.bit_length() - 1         # index of the highest set bit
```

Note `a | b` on ints is **the same operator** as `a | b` on sets. When you convert
set code to mask code, the union line usually does not change at all.

### Operator precedence — the bug that will bite you

In Python, `&` and `|` bind **looser than `==`**. So this is wrong:

```python
if mask & (1 << i) == 0:      # parses as  mask & ((1 << i) == 0)   -> WRONG
```

Always parenthesize the mask test, or use the shift form which is safe because
`>>` binds tighter than `&`:

```python
if (mask & (1 << i)) == 0:    # correct
if not (mask >> i) & 1:       # correct, and reads nicely
```

### Other things that bite

- `1 << i` with a huge `i` is legal in Python (arbitrary precision ints, no overflow)
  but it is not free — do not use bitmasks for `n = 10^5`.
- `~x` in Python is `-x - 1`, i.e. infinite two's complement, so `~mask` alone is
  negative. `mask &= ~(1 << i)` is still correct — only use `~` inside an `&`.
- Off by one on the full mask: `(1 << n) - 1`, **not** `1 << n`.
- Do not confuse the **mask** (which elements) with the **index** (which element).
  `1 << i` converts index → mask; `.bit_length() - 1` converts a single-bit mask → index.

## Walking the set bits

Two idioms. The first is O(n) regardless of how full the mask is:

```python
for i in range(n):
    if (mask >> i) & 1:
        ...  # i is in the set
```

The second is O(popcount) — only touches the bits that are actually set, which
matters inside a hot DP loop:

```python
m = mask
while m:
    low = m & -m               # isolate lowest set bit
    i = low.bit_length() - 1   # its index
    ...                        # i is in the set
    m ^= low                   # clear it and continue
```

`m & -m` works because of two's complement: negating flips all bits above the
lowest set bit and leaves that bit alone, so the AND keeps exactly one bit.

## Pattern 01: One Set Per Node [Reachability]

The most common non-DP use. You are storing "a set of nodes" for every node —
replace each set with an int and every union gets ~free.

Problem: https://leetcode.com/problems/course-schedule-iv

`dfs(x)` returns the set of everything reachable from `x`, including `x` itself.

```python
class Solution:
    def checkIfPrerequisite(self, numCourses, prerequisites, queries):
        adj_list = defaultdict(list)
        for x, y in prerequisites:
            adj_list[x].append(y)

        @cache
        def dfs(parent) -> int:          # <- returns an INT, not a set
            res = 1 << parent            # was: res = {parent}
            for child in adj_list[parent]:
                res |= dfs(child)        # was: res = res | dfs(child)
            return res

        return [bool((dfs(x) >> y) & 1) for x, y in queries]
# The graph is guaranteed a DAG, so no cycle guard is needed.
# @cache gives NO cycle protection on its own -> a cycle here is infinite recursion,
# because the memo is only written AFTER the call returns.
# Time:  O(V + E)  -- each merge is one OR instead of an O(V) set copy
# Space: O(V) ints -- was O(V^2) hash sets
```

Trace it on `prerequisites = [[0,1],[1,2],[0,3]]`, i.e. `adj = {0:[1,3], 1:[2]}`:

```text
dfs(2) = 1<<2                       = 0100 = 4
dfs(3) = 1<<3                       = 1000 = 8
dfs(1) = 1<<1 | dfs(2) = 0010|0100  = 0110 = 6
dfs(0) = 1<<0 | dfs(1) | dfs(3)
       = 0001 | 0110 | 1000         = 1111 = 15

query (0,2) -> (15 >> 2) & 1 = 1 -> True
query (3,1) -> ( 8 >> 1) & 1 = 0 -> False
```

**Honesty note on the complexity.** Python ints are arbitrary precision, so an OR
over a 100-bit value is a couple of machine words, not literally one instruction.
The real cost is `O(V / w)` where `w` is the word size. For `V <= 100` that is a
constant of 2 and nobody cares — but say `O(V/w)` out loud if an interviewer
pushes, it shows you know this is a constant-factor win on the same asymptotic
term, not magic.

**When this stops working.** At `V = 10^5`, one mask is 10^5 bits and you need one
per node — 10^10 bits of precomputed answer. Not storable. At that size you drop
precomputation entirely and run a traversal per query, O(V+E) each. State the
trade-off explicitly: O(1) queries with impossible memory, versus cheap memory
with expensive queries.

### Same pattern, different flavour: mask as a character set

Problem: https://leetcode.com/problems/maximum-product-of-word-lengths

26 lowercase letters is a fixed universe of 26 — perfect. "Do these two words
share a letter?" becomes one AND.

```python
class Solution:
    def maxProduct(self, words: List[str]) -> int:
        masks = []
        for w in words:
            m = 0
            for ch in w:
                m |= 1 << (ord(ch) - ord('a'))
            masks.append(m)

        best = 0
        for i in range(len(words)):
            for j in range(i + 1, len(words)):
                if masks[i] & masks[j] == 0:          # disjoint letter sets
                    best = max(best, len(words[i]) * len(words[j]))
        return best
# Time: O(total_chars + n^2), the pair check is O(1) instead of O(26) set work
```

## Pattern 02: Enumerate All Subsets

`range(1 << n)` walks all 2^n subsets. This is the iterative alternative to the
backtracking subset generator (see [backtrack.md](backtrack.md)) — no recursion,
no state restore, and the mask itself doubles as an index.

Problem: https://leetcode.com/problems/subsets

```python
class Solution:
    def subsets(self, nums: List[int]) -> List[List[int]]:
        n = len(nums)
        res = []
        for mask in range(1 << n):              # 0 .. 2^n - 1
            current = [nums[i] for i in range(n) if (mask >> i) & 1]
            res.append(current)
        return res
# Time: O(n * 2^n)   Space: O(n * 2^n) for the output
```

Every subset gets a unique integer name, which is the real payoff — that name is
what you memoize on in the next pattern.

### Enumerating submasks of a mask

Occasionally you need "every subset of *this* subset" (set-cover style problems).
The idiom is non-obvious enough to just memorize:

```python
sub = mask
while sub:
    ...                       # sub is a non-empty submask of mask
    sub = (sub - 1) & mask    # step to the next one down
# the empty submask (0) is never visited by this loop -- handle it separately
```

Subtracting 1 borrows through the low zeros; the `& mask` snaps you back inside
the parent set. Over all masks this is O(3^n) total, not O(4^n).

## Pattern 03: Bitmask DP [subset as a DP state]

This is where bitmasks earn their keep. The state is "which elements have I used
so far", which is a **set** — and a set cannot be a dict key or an `@cache` key.
As an int, it can.

Shape: `dp(mask)` = best answer given that `mask` is already handled.

Canonical example — assign `n` tasks to `n` workers, minimum total cost. The
trick is that `popcount(mask)` tells you *which task you are on*, so the task
index does not need to be a separate state dimension.

```python
def min_assignment_cost(cost: list[list[int]]) -> int:
    n = len(cost)
    full = (1 << n) - 1

    @cache
    def dp(mask: int) -> int:
        i = mask.bit_count()          # tasks 0..i-1 are done -> now assigning task i
        if mask == full:
            return 0
        best = inf
        for j in range(n):
            if not (mask >> j) & 1:               # worker j still free
                best = min(best, cost[i][j] + dp(mask | (1 << j)))
        return best

    return dp(0)
# States: 2^n. Transitions per state: n. -> Time O(n * 2^n), Space O(2^n)
# For n = 20 that is ~2e7, fine. For n = 25 it is not.
```

The complexity `O(n * 2^n)` is the signature of this pattern. If the constraints
say `n <= 20` (or 15, or 12) and the problem smells exponential, bitmask DP is
almost certainly the intended solution — that small `n` is the interviewer
telling you exponential-in-n is acceptable.

Bottom-up is the same recurrence over `for mask in range(1 << n)`, and it is safe
because `mask | (1 << j) > mask` always, so masks are naturally processed in a
valid dependency order when you iterate upward.

## Pattern 04: State Compression in BFS

Same idea as above, but the mask rides along in the BFS queue as part of the
state. The visited set becomes `(node, mask)` rather than just `node` — you may
legitimately revisit a node with a different set of things collected.

Problem: https://leetcode.com/problems/shortest-path-visiting-all-nodes

```python
class Solution:
    def shortestPathLength(self, graph: List[List[int]]) -> int:
        n = len(graph)
        full = (1 << n) - 1

        # Multi-source: we may start anywhere (see bfs.md)
        que = deque((i, 1 << i, 0) for i in range(n))
        seen = {(i, 1 << i) for i in range(n)}

        while que:
            node, mask, dist = que.popleft()
            if mask == full:                     # collected every node
                return dist
            for child in graph[node]:
                nmask = mask | (1 << child)
                if (child, nmask) not in seen:   # state = WHERE + WHAT I HAVE
                    seen.add((child, nmask))
                    que.append((child, nmask, dist + 1))
        return 0
# States: n * 2^n. Time O(n^2 * 2^n) with the edge loop.
# n = 1 works for free: the initial mask is already full, dist 0.
```

The tell for this pattern: the problem lets you revisit nodes, and asks about
collecting/visiting a set of things. Plain BFS `visited` marks a node done
forever and gives a wrong answer; the mask is what makes revisits legal.

A grid variant of the same idea is "keys and doors" — the mask is which keys you
hold, and a door is passable only if `(keys >> door) & 1`.

## Not the same thing: bit tricks

Bit *manipulation* puzzles (XOR to find the single number, counting set bits,
power-of-two checks, swapping without a temp) share the operators but not the
idea. This note is only about **an int standing in for a set**. Keep them
separate in your head or you will reach for the wrong tool.

The one overlap worth knowing: `a ^ b ^ b == a`, which is why XOR cancels pairs.

## MUST DO while revisiting problems

**Set-per-node / mask as a set**

* https://leetcode.com/problems/course-schedule-iv/  (reachability, the worked example above)
* https://leetcode.com/problems/maximum-product-of-word-lengths/  (26-letter mask, disjointness)

**Subset enumeration**

* https://leetcode.com/problems/subsets/  (iterative `range(1 << n)`)
* https://leetcode.com/problems/beautiful-arrangement/  (mask + popcount as position)

**Bitmask DP**

* https://leetcode.com/problems/partition-to-k-equal-sum-subsets/  (mask of used elements)
* https://leetcode.com/problems/can-i-win/  (memo on the mask of taken numbers)
* https://leetcode.com/problems/smallest-sufficient-team/  (skills as a mask, hard)
* https://leetcode.com/problems/find-the-shortest-superstring/  (TSP in disguise, hard)
* https://leetcode.com/problems/maximum-students-taking-exam/  (one mask per grid row, hard)

**State compression search**

* https://leetcode.com/problems/shortest-path-visiting-all-nodes/  (BFS on `(node, mask)`)
* https://leetcode.com/problems/shortest-path-to-get-all-keys/  (grid + key mask, hard)