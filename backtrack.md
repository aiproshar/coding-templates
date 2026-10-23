 # Backtrack Basics
 
There is basically one single framework, CCG: Choices, Constraints, Goals

Choices → Where can we go from current state

Constraints → Are we jumping into a valid state space ? What constraints we have ? For example out of bound in a 2D grid and already visited nodes

Goals → What we want to achieve ? For example in word search on 2D matrix our goal is can we find the word ?

#### This is called the CCG framework, and this is standard. Lets see how this unfolds on a pseudocode

```python
def backtrack(state, ...):
    # 1. BASE CASE (Success / Stop condition) -> Goals
    if is_solution(state):
        # Note: If 'state' is a mutable list, append a COPY of it to results!
        results.append(list(state)) 
        return
    # invalid case, we don't want to move forward anymore
    if invalid(state): # Constraints, we dont move forward
        return

    # 2. EXPLORE CHOICES
    for choice in get_available_choices(state):
        if is_valid(choice):
            
            # --- A. STATE UPDATE ---
            state.append(choice)
            
            # --- B. RECURSE ---
            backtrack(state, ...)
            
            # --- C. UNDO STATE (The "Backtrack") ---
            state.pop()

```
### A bit more non-generic example, 2D grid traversal, this is the generic template
```python
def backtrack_grid(grid, x, y, path):
    # 1. CAPTURE ORIGINAL STATE (Defensive habit)
    original_val = grid[x][y]
    
    # 2. STATE UPDATE (Mark as visited, build path)
    path.append(original_val)
    grid[x][y] = VISITED 
    
    # 3. PROCESS CURRENT STATE
    if meets_condition(path):
        results.append(list(path))
        
    # 4. RECURSE (Check bounds and validity BEFORE calling)
    directions = [(0, 1), (1, 0), (0, -1), (-1, 0)]
    for dx, dy in directions:
        nx = x + dx
        ny = y + dy
        
        # --- BOUNDARY & VALIDITY CHECK ---
        if 0 <= nx < len(grid) and 0 <= ny < len(grid[0]):
            if grid[nx][ny] != VISITED:
                # Only call if the neighbor is strictly valid
                backtrack_grid(grid, nx, ny, path)
            
    # 5. UNDO STATE (Restore path and grid exactly as they were)
    path.pop()
    grid[x][y] = original_val
```


## Types 01: Generate All Subsets from n distinct/with duplicate elements


General idea, at each position we have two options, take it or skip it.
```python
class Solution:
    def subsets(self, nums: List[int]) -> List[List[int]]:

        res = []

        def backtrack(i, current) -> None:
            if i >= len(nums):
                res.append(current.copy())
                return
            
            # Take ith element
            current.append(nums[i])
            backtrack(i+1, current)

            # backtrack
            current.pop()

            # Skip ith element
            backtrack(i+1, current)
            return
        
        backtrack(0, [])
        return res
```
At each depth, we only think about a specific element, we add it and skip it in all
branches that reaches that depth

Here is the viz for [1,2,3]
```text
([], i=0)
├── take 1 → ([1], i=1)
│   ├── take 2 → ([1,2], i=2)
│   │   ├── take 3 → ([1,2,3], i=3)  → res += [1,2,3]
│   │   └── skip 3 → ([1,2], i=3)    → res += [1,2]
│   └── skip 2 → ([1], i=2)
│       ├── take 3 → ([1,3], i=3)    → res += [1,3]
│       └── skip 3 → ([1], i=3)      → res += [1]
└── skip 1 → ([], i=1)
    ├── take 2 → ([2], i=2)
    │   ├── take 3 → ([2,3], i=3)    → res += [2,3]
    │   └── skip 3 → ([2], i=3)      → res += [2]
    └── skip 2 → ([], i=2)
        ├── take 3 → ([3], i=3)      → res += [3]
        └── skip 3 → ([], i=3)       → res += []
```

Another method is take from the available rest
```python
```python3
class Solution:
    def subsets(self, nums: List[int]) -> List[List[int]]:
        
        subsets = []
        def backtrack(start, current):
            subsets.append(current.copy())
            # take any of the future elements
            for i in range(start, len(nums)):
                current.append(nums[i])
                backtrack(i+1, current)
                current.pop()
        backtrack(0, [])
        return subsets
```
Here is the state space tree
```text
([], i=0)
├── take 1 → ([1], i=1)
│   ├── take 2 → ([1,2], i=2)
│   │   ├── take 3 → ([1,2,3], i=3)  → res += [1,2,3]
│   │   └── skip 3 → ([1,2], i=3)    → res += [1,2]
│   └── skip 2 → ([1], i=2)
│       ├── take 3 → ([1,3], i=3)    → res += [1,3]
│       └── skip 3 → ([1], i=3)      → res += [1]
└── skip 1 → ([], i=1)
    ├── take 2 → ([2], i=2)
    │   ├── take 3 → ([2,3], i=3)    → res += [2,3]
    │   └── skip 3 → ([2], i=3)      → res += [2]
    └── skip 2 → ([], i=2)
        ├── take 3 → ([3], i=3)      → res += [3]
        └── skip 3 → ([], i=3)       → res += []
```
visual for this one, normal way the first response like this

### Todo:
- https://leetcode.com/problems/subsets/

Additional do it: subset with duplicates, the sort and skip if matches backward technique
Problem: https://leetcode.com/problems/subsets-ii/description/



## Type 02: Combination
Combination is a subset, where each subset is a specific length. For example from [1,2,3,4] if we 
take 2 elements, it can be 1,2/2,3/3,4/1,3/1,4/2,4 etc 
From [1,2,3,4] if we take all we will have only 1 combination, [1,2,3,4]
nCr = n!/ {(n-r)!r!}  [pick r elements from n]

Combination is similar to subset, subsets generate from 0 to N len combination, but in 
combination we just generate a specific len. So we can add a current len check and append and terminate early
```python
class Solution:
    def combine(self, n: int, k: int) -> List[List[int]]:
        res = []
        def backtrack(i, current):
            if len(current) == k:
                res.append(current.copy())
                return
            # Two exit case
            # one obvious: out of bound
            # second: how many elements we need and we also have
            # if we need 5 elements and have 4 whatever way we go we dont get k elements
            # Off by one corner case, in I state we have not decided to take /skip i yet
            # so n - i + 1 is inclusive how many we need to have k elements
            # (+1) because we haven't taken for its position yet
            if i > n or n - i  + 1 < k - len(current):
                return
            # take ith element
            current.append(i)
            backtrack(i+1, current)
            current.pop()
            # skip ith element
            backtrack(i+1, current)
        backtrack(1, [])
        return res
```
- Todo: https://leetcode.com/problems/combinations/description/

Additional: Combination Sum: Instead of taking specific sized subsets, we take the subsets
when they add up to specific target, also keep track of current sum for lower complexity
Also handle duplicate case by forward move
- https://leetcode.com/problems/combination-sum
- https://leetcode.com/problems/combination-sum-ii

##### Example Solution
```python
# Take it or leave it approach
class Solution:
    def combinationSum(self, candidates: List[int], target: int) -> List[List[int]]:
        
        res = set()

        def backtrack(i, sum_, current):
            if sum_ == target:
                res.add(tuple(current))
                return
            if i >= len(candidates) or sum_ > target:
                return
            
            # take ith element
            current.append(candidates[i])
            backtrack(i, sum_ + candidates[i], current)
            current.pop()

            # skip ith element
            backtrack(i+1, sum_, current)
        backtrack(0, 0, [])
        return [list(x) for x in res]
# Take one for all next available, better time complexity
class Solution2:
    def combinationSum(self, candidates: List[int], target: int) -> List[List[int]]:
        candidates.sort()
        res = []

        def backtrack(start, sum_, current):
            if sum_ == target:
                res.append(current.copy())
                return
            if sum_ > target:
                return
            for i in range(start, len(candidates)):
                if sum_ + candidates[i] > target:
                    break
                current.append(candidates[i])
                backtrack(i, sum_ + candidates[i], current)
                current.pop()
            return
        backtrack(0, 0, [])
        return res
```
## Backtrack on 2D Grid
- https://leetcode.com/problems/word-search/description/
## Additional Types
- https://leetcode.com/problems/partition-to-k-equal-sum-subsets/
- https://leetcode.com/problems/n-queens/description/
- https://leetcode.com/problems/letter-combinations-of-a-phone-number/description/

## Super Hard
- https://leetcode.com/problems/word-break-ii/description/ [DP, we cannot pass current state, build up state as we return,
but also solvable via raw backtrack and state forward propagation, but TLE]

