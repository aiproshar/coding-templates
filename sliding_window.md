# Sliding Window

One single template to rule em all, extend window. Then keep shrinking until window is invalid

### Variable Length Window
This is variable window, we grow until its invalid, then continue shrink until its valid again
```python
i = 0
j = 0
# i is left window start position
# j is right window end position
global_max = 0
for j in range(len(nums)):
    
    # modify state to include array[i], can be anything. freq, sum ect
    state_update(nums[j])
    
    # if including nums[j] made the state invalid, start shrinking
    #  until our window is valid again
    while (state is not valid):
        # undo operation
        state_remove(nums[i])
        i += 1
    # Now we are in valid state, do the global update whatever we're looking for
    global_max = max(global_max, current_state_value)
    # max could be min or even anything, or simply trying to see if state can reach some constant
    
    

```


### Fixed Window length

For a fixed window, our window size is always constant, and its the first condition.
ALways check j - i + 1 == k first, if we grown past K, reduce


```python
i = 0
global_max = 0

for j in range(len(nums)):

    # 1. add the new right element to the state
    state_update(nums[j])

    # 2. window size is j - i + 1. If it exceeds k, drop the left element
    if j - i + 1 > k:
        state_remove(nums[i])
        i += 1
    
    # skipping initial windows  < k while we are growing
    # 3. only record an answer once the window has reached size k
    if j - i + 1 == k:
        global_max = max(global_max, current_state_value)
```

### Some Things to Consider

- Modify the input array if necessary, for example we can sort
- Upon exiting from invalid state, sometimes we need to do additional check. For example specific size window
- Some Problems require to update to invalid state (LC 209 )


### Todo Problems:

#### Basic Straight Forward
- https://leetcode.com/problems/longest-substring-without-repeating-characters/description/
- https://leetcode.com/problems/max-consecutive-ones-iii/
- https://leetcode.com/problems/permutation-in-string/description/


# Hard Variants [Must do]
- https://leetcode.com/problems/fruit-into-baskets/description/
- https://leetcode.com/problems/sliding-window-maximum/description/
- https://leetcode.com/problems/minimum-window-substring/description/
- https://leetcode.com/problems/frequency-of-the-most-frequent-element/description/ (Weird math in state calculation)
- https://leetcode.com/problems/minimum-size-subarray-sum/description/ (Optimize in invalid State)
- https://leetcode.com/problems/subarray-product-less-than-k/description/ (weird math in state calculation)