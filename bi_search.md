# Binary Search — Three Core Templates

Binary search is O(log n). The idea is never the hard part — getting the loop
condition, the boundary updates, and the return value exactly right is. Three
templates cover almost everything.

One example array throughout, so you can see how each template behaves on the
**same** input:

```
    nums  = [1, 3, 5, 5, 5, 7, 9]
    index    0  1  2  3  4  5  6

    target = 5   (lives at indices 2, 3, 4)
```

- **Template 1 (exact):**      returns `3`  — any 5
- **Template 2 (left bound):** returns `2`  — first 5
- **Template 3 (right bound):** returns `4` — last 5

Same array, same target, three different answers. They only differ in what they
do when they **find** a match:

- exact → return immediately
- left bound → keep searching left (`hi = mid`)
- right bound → keep searching right (`lo = mid`, mid rounded up)

## Template 1 — Exact Match

Find ANY index of a specific value, or `-1` if absent.

```python
def binary_search_exact(nums: list[int], target: int) -> int:
    lo = 0
    hi = len(nums) - 1

    while lo <= hi:                 # space can shrink to one element
        mid = (lo + hi) // 2

        if nums[mid] == target:
            return mid              # found it, done
        elif nums[mid] < target:
            lo = mid + 1            # target is right, exclude mid
        else:
            hi = mid - 1            # target is left, exclude mid

    return -1                       # exhausted, not found
```

- Loop is `lo <= hi` — the last candidate is when `lo == hi`, so we must still
  check it.
- Both bounds exclude `mid` (`mid + 1`, `mid - 1`), and there is an early return
  on match.

## Template 2 — Left Bound  (lower bound / find minimum)

Find the LEFTMOST index that satisfies a condition. Classic use: first
occurrence of target.

```python
def binary_search_left_bound(nums: list[int], target: int) -> int:
    lo = 0
    hi = len(nums)                  # NOTE: len(nums), not len(nums) - 1
    # if every element is < target, lo converges to len(nums) = "insert at end"

    while lo < hi:                  # strict! converges when lo == hi
        mid = (lo + hi) // 2

        if nums[mid] >= target:     # condition met: mid COULD be the answer
            hi = mid                # keep mid, search left
        else:
            lo = mid + 1            # too small, skip mid

    if lo < len(nums) and nums[lo] == target:
        return lo
    return -1
```

- **`hi = mid`, not `mid - 1`** — mid might be the answer, do not throw it away.
- **`lo < hi`, not `lo <= hi`** — with `hi = mid`, using `<=` infinite-loops
  when `lo == hi == mid`. Strict `<` terminates at the convergence point.
- The `==` case is NOT special — "equals target" and "condition met" are the
  same branch (both shrink `hi`).

## Template 3 — Right Bound  (find maximum)

Find the RIGHTMOST index that satisfies a condition. Classic use: last
occurrence of target.

```python
def binary_search_right_bound(nums: list[int], target: int) -> int:
    if not nums:
        return -1

    lo = 0
    hi = len(nums) - 1

    while lo < hi:
        mid = (lo + hi + 1) // 2    # round UP — see below

        if nums[mid] <= target:     # condition met: mid COULD be the answer
            lo = mid                # keep mid, search right
        else:
            hi = mid - 1            # too big, skip mid

    if nums[lo] == target:
        return lo
    return -1
```

- **`mid = (lo + hi + 1) // 2` — round UP.** This is the #1 gotcha. When
  `lo + 1 == hi` and the condition is met, we do `lo = mid`. If `mid` rounds
  down it equals `lo`, so `lo = lo` → infinite loop. Rounding up makes
  `mid == hi`, so `lo = hi` and the loop ends.
- Mirror image of left bound: on a match we set `lo = mid` and keep going right.

## Cheat Sheet

```
  Template       Loop       mid formula        On match      On miss        Return
  ─────────────  ─────────  ─────────────────  ────────────  ─────────────  ──────
  Exact match    lo <= hi   (lo + hi) // 2     return mid    lo=mid+1 /     -1
                                                             hi=mid-1
  Left bound     lo < hi    (lo + hi) // 2     hi = mid      lo = mid + 1   lo
  (find min)
  Right bound    lo < hi    (lo + hi + 1) // 2 lo = mid      hi = mid - 1   lo
  (find max)                ↑ round up!
```

## Decision Guide

1. Looking for a specific value? → **Template 1** (exact).
2. Looking for the SMALLEST value satisfying a condition? → **Template 2**
   (left bound): `hi = mid`, mid rounds down.
3. Looking for the LARGEST value satisfying a condition? → **Template 3**
   (right bound): `lo = mid`, mid rounds **up**.

When the target is missing, left bound converges onto the first element `>`
target (`bisect_left` — "where would it be inserted?") and right bound onto the
last element `<` target (predecessor query). Neither tells you it is missing —
check `nums[lo]` afterward if you need an exact match.

## Common Pitfalls

1. **Off-by-one in `hi` init** — exact & right bound use `len(nums) - 1`, left
   bound uses `len(nums)` (so "not found" can converge past the end).
2. **Forgetting to round up in right bound** → infinite loop when `lo + 1 == hi`.
3. **Using `lo <= hi` with `hi = mid`** → infinite loop when `lo == hi == mid`.
4. **Early return in the bound templates** → returns the first match found, not
   the actual min/max. Only Template 1 returns early.

## Python's `bisect` — don't hand-roll it

The standard library already ships Templates 2 and 3 as `bisect`. In an
interview or real code, reach for this first — it is C-fast and bug-free.

### The one idea: the optimal insertion point

`bisect` really does **one** thing: given a sorted list and a value `x`, it
tells you *the single index where `x` belongs so the list stays sorted*. That
index is the "insertion point", and it is the whole purpose of the module — the
name is literally short for "binary bisection to find where to insert".

Think of a sorted row of theatre seats already filled by height, shortest on the
left. A new person walks in. There is exactly **one** gap they can slip into
without anyone having to swap seats — just left of the first person taller than
them. `bisect` finds that gap in O(log n) by repeatedly halving the row instead
of walking it person by person. It never actually seats anyone (that is
`insort`'s job); it only points at the gap.

```
    heights = [150, 160, 165, 172, 180]
    new person = 168 cm

    150   160   165   |   172   180
     0     1     2    ↑    3     4
                  insertion point = 3
                  "168 goes here — right after 165, before 172"
```

Here is the payoff, and why the rest of this section is so short: **every other
query is just this one number, read differently.**

- Want the *count of elements `< x`*? That is the insertion point itself —
  everything to its left is smaller.
- Want to *find* `x`? Look at the insertion point; if `nums[i] == x`, it is
  there, else it is absent.
- Want the *floor* (largest `<= x`)? Step one left of the insertion point.
- Want to *insert and stay sorted*? Drop `x` into the insertion point.

So you are never really "searching" — you are asking "where does this belong?"
and then interpreting that gap. `bisect_left` and `bisect_right` are just two
flavours of the same gap: when duplicates of `x` already exist, `_left` gives
the gap *before* them, `_right` gives the gap *after* them.

> **Gotcha: `bisect` never tells you whether `x` exists.** It always returns an
> insertion point — a valid index in `0..len(nums)` — even when `x` is absent,
> because *some* gap always exists. `bisect_left(nums, 6)` on `[1,3,5,5,5,7,9]`
> returns `5`, the same index it would return if a `6` were actually there. So
> the returned index is "where `x` would go", **not** "where `x` is". If you
> need an exact-match answer, you must check it yourself:
>
> ```python
> i = bisect.bisect_left(nums, x)
> found = i < len(nums) and nums[i] == x    # <-- YOU must verify
> ```
>
> Two things to guard: `i` can equal `len(nums)` (x is bigger than everything,
> so indexing `nums[i]` would go out of bounds — check `i < len(nums)` first),
> and even in-bounds, `nums[i]` might simply not equal `x`. This is exactly the
> `if lo < len(nums) and nums[lo] == target` tail from Template 2.

```python
import bisect

nums = [1, 3, 5, 5, 5, 7, 9]
#        0  1  2  3  4  5  6

bisect.bisect_left(nums, 5)    # -> 2   leftmost slot for 5  (== Template 2)
bisect.bisect_right(nums, 5)   # -> 5   rightmost slot for 5 (one past last 5)
bisect.bisect(nums, 5)         # -> 5   alias for bisect_right
```

Both return an **insertion index** that keeps the list sorted. The difference is
only how they treat equals: `bisect_left` lands *before* existing copies,
`bisect_right` lands *after* them. When the value is absent they return the same
thing — the gap where it would go:

```python
bisect.bisect_left(nums, 6)    # -> 5   6 would be inserted at index 5
bisect.bisect_left(nums, 0)    # -> 0   smaller than everything
bisect.bisect_left(nums, 10)   # -> 7   bigger than everything (== len)
```

`insort` inserts while keeping the list sorted (`bisect` + `list.insert`), O(n)
because of the shift:

```python
arr = [1, 3, 5, 7]
bisect.insort(arr, 4)          # arr -> [1, 3, 4, 5, 7]
```

All four take optional `lo`, `hi` bounds to restrict the search window, and a
`key=` function (Python 3.10+) so you can bisect a list of objects.

### Common use cases

```python
# 1. First / last index of a target (or -1 if absent)
def first(nums, x):
    i = bisect.bisect_left(nums, x)
    return i if i < len(nums) and nums[i] == x else -1

def last(nums, x):
    i = bisect.bisect_right(nums, x) - 1
    return i if i >= 0 and nums[i] == x else -1

# 2. Count occurrences of x in a sorted list — O(log n)
count = bisect.bisect_right(nums, x) - bisect.bisect_left(nums, x)

# 3. Count how many elements are < x, <= x, > x, etc.
less_than       = bisect.bisect_left(nums, x)          # elements strictly < x
less_or_equal   = bisect.bisect_right(nums, x)         # elements <= x

# 4. Floor / predecessor (largest element <= x), or -1
def floor(nums, x):
    i = bisect.bisect_right(nums, x)
    return nums[i - 1] if i else -1

# 5. Ceil / successor (smallest element >= x), or -1
def ceil(nums, x):
    i = bisect.bisect_left(nums, x)
    return nums[i] if i < len(nums) else -1

# 6. Map a score to a bucket / grade
grades = [60, 70, 80, 90]
letters = "FDCBA"
def grade(score):
    return letters[bisect.bisect_right(grades, score)]   # 85 -> 'B'
```

Rule of thumb: `bisect_left(a, x)` = "how many are strictly less than `x`" and
also "leftmost place `x` fits"; `bisect_right(a, x)` = "how many are `<= x`".
Almost every sorted-array counting question reduces to those two.

## MUST DO while revisit problems

* https://leetcode.com/problems/binary-search/  (exact match)

* https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/  (left + right bound)

* https://leetcode.com/problems/search-insert-position/  (left bound / bisect_left)
