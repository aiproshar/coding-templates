# Binary Search — One Template

Binary search is O(log n). The idea is never the hard part — getting the loop
condition, the boundary updates, and the return value exactly right is. This
note uses **a single half-open skeleton** for everything, so there is only one
set of rules to remember.

One example array throughout, so you can see how each query behaves on the
**same** input:

```
    nums  = [1, 3, 5, 5, 5, 7, 9]
    index    0  1  2  3  4  5  6

    target = 5   (lives at indices 2, 3, 4)
```

- **exact:**       returns `2` (or -1 if absent) — via left bound + a check
- **left bound:**  returns `2`  — first 5
- **right bound:** returns `4`  — last 5

Same array, same target, different answers — but **the same loop**. They only
differ in **one comparison operator** and how you read the result.

## The Skeleton — memorize this once

```python
def bound(nums: list[int], target: int) -> int:
    lo, hi = 0, len(nums)          # half-open range [lo, hi)  — hi = len, not len-1
    while lo < hi:                 # strict <  — converges when lo == hi
        mid = (lo + hi) // 2       # always round DOWN
        if <go-right condition>:   # mid is too small / still ok — discard it
            lo = mid + 1
        else:                      # mid could be the answer — keep it
            hi = mid
    return lo                      # insertion point, always in [0, len(nums)]
```

Four fixed rules, never vary:

1. **`hi = len(nums)`** — half-open. Lets the answer converge to `len` ("insert
   at end") instead of being capped at the last index.
2. **`while lo < hi`** — strict. With `hi = mid`, using `<=` infinite-loops when
   `lo == hi == mid`.
3. **`mid = (lo + hi) // 2`** — always round down. No `+1` gotcha, because the
   `keep-mid` branch is `hi = mid` (which always shrinks), never `lo = mid`.
4. **Updates are always `lo = mid + 1` / `hi = mid`** — the go-right branch
   excludes mid, the keep branch retains it.

Everything below is this skeleton with a different `<go-right condition>` and a
different way of reading `lo`.

## Left Bound (lower bound / `bisect_left`)

First index where `nums[i] >= target`. Go right while `mid` is strictly too
small.

```python
def left_bound(nums: list[int], target: int) -> int:
    lo, hi = 0, len(nums)
    while lo < hi:
        mid = (lo + hi) // 2
        if nums[mid] < target:      # strictly less  -> answer is to the right
            lo = mid + 1
        else:                       # >= target      -> mid could be first, keep
            hi = mid
    return lo
```

- Returns the leftmost slot for `target`, or where it would be inserted if absent.
- This is exactly `searchInsert` / `bisect_left`.

## Right Bound (upper bound / `bisect_right`)

First index where `nums[i] > target`. **Identical to left bound except `<`
becomes `<=`.** That single operator is the entire difference between the two
bounds.

```python
def right_bound(nums: list[int], target: int) -> int:
    lo, hi = 0, len(nums)
    while lo < hi:
        mid = (lo + hi) // 2
        if nums[mid] <= target:     # <= target  -> answer is to the right
            lo = mid + 1
        else:                       # > target   -> mid could be first-greater, keep
            hi = mid
    return lo
```

- Returns one **past** the last `target` (the gap after all copies).
- Last occurrence of `target` is therefore `right_bound(nums, target) - 1`.

## Exact Match

No separate template needed — it is left bound plus a verification tail. The
`while` loop finds where `target` *would* go; then you check whether it is
actually there.

```python
def exact(nums: list[int], target: int) -> int:
    lo, hi = 0, len(nums)
    while lo < hi:
        mid = (lo + hi) // 2
        if nums[mid] < target:
            lo = mid + 1
        else:
            hi = mid
    # lo is now the insertion point — verify a real match sits there
    if lo < len(nums) and nums[lo] == target:
        return lo
    return -1
```

- The two guards matter: `lo` can equal `len(nums)` (target bigger than
  everything, so `nums[lo]` would be out of bounds — check `lo < len(nums)`
  first), and even in-bounds `nums[lo]` might simply not equal `target`.

## Cheat Sheet

```
  Query          go-right condition     Return                     Notes
  ─────────────  ─────────────────────  ─────────────────────────  ──────────────────
  Left bound     nums[mid] <  target    lo                         bisect_left
  Right bound    nums[mid] <= target    lo                         bisect_right
  Exact match    nums[mid] <  target    lo if nums[lo]==target      left bound + check
                                        else -1

  ALL THREE share:  lo,hi = 0,len(nums) · while lo < hi · mid=(lo+hi)//2 (round down)
                    lo = mid + 1  /  hi = mid
```

The only knob is `<` vs `<=`. Exact match reuses `<` and adds a check.

## Decision Guide

1. Need a specific value's index? → **exact** (left bound, then verify `nums[lo]`).
2. Need the leftmost position where a condition first holds? → **left bound**
   (`<` in the go-right test).
3. Need one-past the rightmost position, or the last occurrence? → **right
   bound** (`<=`), then subtract 1 for the last index.

When the target is missing, left and right bound return the **same** index — the
single gap where it would be inserted. They can only diverge when at least one
copy is present; the size of the gap between them is exactly the count of copies.

## Derived queries (all from the two bounds)

```python
count          = right_bound(nums, x) - left_bound(nums, x)   # occurrences of x
first_index    = left_bound(nums, x)                          # if nums[i]==x
last_index     = right_bound(nums, x) - 1                     # if valid & ==x
num_less_than  = left_bound(nums, x)                          # elements < x
num_less_equal = right_bound(nums, x)                         # elements <= x
```

## Python's `bisect` — don't hand-roll it

The standard library already ships both bounds as `bisect`. In an interview or
real code, reach for this first — it is C-fast and bug-free.

### The one idea: the optimal insertion point

`bisect` really does **one** thing: given a sorted list and a value `x`, it
tells you *the single index where `x` belongs so the list stays sorted*. That
index is the "insertion point", and it is the whole purpose of the module — the
name is literally short for "binary bisection to find where to insert".

```
    heights = [150, 160, 165, 172, 180]
    new person = 168 cm

    150   160   165   |   172   180
     0     1     2    ↑    3     4
                  insertion point = 3
                  "168 goes here — right after 165, before 172"
```

Every other query is just this one number, read differently:

- Count of elements `< x`? The insertion point itself.
- Find `x`? Look at the insertion point; if `nums[i] == x` it is there, else absent.
- Floor (largest `<= x`)? Step one left of the insertion point.
- Insert and stay sorted? Drop `x` into the insertion point.

`bisect_left` and `bisect_right` are just two flavours of the same gap: when
duplicates of `x` exist, `_left` gives the gap *before* them, `_right` the gap
*after*.

> **Gotcha: `bisect` never tells you whether `x` exists.** It always returns an
> insertion point — a valid index in `0..len(nums)` — even when `x` is absent,
> because *some* gap always exists. `bisect_left(nums, 6)` on `[1,3,5,5,5,7,9]`
> returns `5`, the same index it would return if a `6` were actually there. So
> the returned index is "where `x` would go", **not** "where `x` is". If you
> need an exact-match answer, verify it yourself — this is exactly the tail in
> the `exact` function above:
>
> ```python
> i = bisect.bisect_left(nums, x)
> found = i < len(nums) and nums[i] == x    # <-- YOU must verify
> ```

```python
import bisect

nums = [1, 3, 5, 5, 5, 7, 9]
#        0  1  2  3  4  5  6

bisect.bisect_left(nums, 5)    # -> 2   leftmost slot for 5  (== left bound)
bisect.bisect_right(nums, 5)   # -> 5   rightmost slot for 5 (== right bound)
bisect.bisect(nums, 5)         # -> 5   alias for bisect_right

bisect.bisect_left(nums, 6)    # -> 5   6 would be inserted at index 5 (absent)
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

# 3. Count how many elements are < x, <= x
less_than     = bisect.bisect_left(nums, x)      # elements strictly < x
less_or_equal = bisect.bisect_right(nums, x)     # elements <= x

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

## Common Pitfalls

1. **Using `len(nums) - 1` for `hi`** — with the half-open skeleton `hi` is
   always `len(nums)` so the answer can converge past the end ("insert at end").
2. **Using `lo <= hi`** — with `hi = mid` this infinite-loops when
   `lo == hi == mid`. Always strict `<`.
3. **Reaching for `lo = mid` / round-up** — you never need it here. The keep
   branch is always `hi = mid`, which always shrinks, so `mid` always rounds
   down. (Round-up is only required in the *inclusive* `[lo, hi]` style, which
   this note avoids entirely.)
4. **Trusting the bound as an exact match** — a bound is always an insertion
   point; verify `nums[lo] == target` (with the `lo < len(nums)` guard) if you
   need existence.

## MUST DO while revisiting problems

* https://leetcode.com/problems/binary-search/  (exact match)
* https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/  (left + right bound)
* https://leetcode.com/problems/search-insert-position/  (left bound / bisect_left)