 # Backtrack Basics
 
There is basically one single framework, CCG: Choices, Constrains, Goals

Choices → Where can we go from current state

Constrains → Are we jumping into a valid state space ? What constrains we have ? For example out of bound in a 2D grid and already visited nodes

Goals → What we want to achieve ? For example in word search on 2D matrix our goal is can we find the word ?

#### This is called the CCG framework, and this is standard. Lets see how this unfolds on a psudocode

```python
def backtrack(state, ...):
    # 1. BASE CASE (Success / Stop condition) -> Goals
    if is_solution(state):
        # Note: If 'state' is a mutable list, append a COPY of it to results!
        results.append(list(state)) 
        return
    # invalid case, we don't want to move forward anymore
    if invalid(state): # Constrains, we dont move forward
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
