## Where Everything Started: LC 912

We need to sort an array ! But we cannot use any built-in functions like below

```python

class Solution:
    def sortArray(self, nums: List[int]) -> List[int]:
        nums.sort()
        return nums
# Built in timsort [stable sorting algorithm that combines merge sort and insertion sort]
# time : O(nlogn) worst case
# Space: O(n) worst case
```

### First Option: Selection Sort [TLE on LC 912]

Pretty Self explanatory, we find the sorted number, swap with current index and move forward
All numbers before current index is sorted, that's the loop invariant
```python
class Solution:
    def sortArray(self, nums: List[int]) -> List[int]:

        for i in range(len(nums)):
            idx = i
            
            for j in range(i+1, len(nums)):
                if nums[j] < nums[idx]:
                    idx = j
            nums[i], nums[idx] = nums[idx], nums[i]
        return nums 
# Time Complexity: O(n^2)
# Space: O(1) Truly in Place  modification
```

### Second Option: Built in heapq load bearing
This is a bit cheating, so ask the interviewer if we are allowed to use python's built in 
HEAP library or not. Dont go this path without consulting with the interviewer

Technically we are handing over the load bearing to heapq library
```python
class Solution:
    def sortArray(self, nums: List[int]) -> List[int]:

        heap = nums.copy()
        heapq.heapify(heap)

        for i in range(len(nums)):
            nums[i] = heap[0]
            heapq.heappop(heap)
        return nums
# Heapify initially takes O(n) time
# Each operation is logn, we do n times and also a second placeholder array
# Time Complexity: O(nlogn)
# Space Complexity: O(n)
```

### Third Option and Safest One: Merge Sort
I was able to write this up in one try, so this is pretty easy and safest to come up in 
a pressure cooker situation.
```python
class Solution:
    def sortArray(self, nums: List[int]) -> List[int]:


        def merge(array) -> list[int]:
            if len(array) <= 1:
                return array
            mid = len(array) // 2
            first = merge(array[:mid])
            second = merge(array[mid:])
            ans = []
            i = j = 0
            while i < len(first) and j < len(second):
                if first[i] < second[j]:
                    ans.append(first[i])
                    i += 1
                else:
                    ans.append(second[j])
                    j += 1
            while i < len(first):
                ans.append(first[i])
                i += 1
            while j < len(second):
                ans.append(second[j])
                j += 1
            return ans
        ans = merge(nums)
        return ans  
# Time Complexity: n log n, we have logn levels depth, each at max n items compare
# space: O(n), not nlogn, because this is DFS, we explore one branch and backtrack
# In DFS, only one root to leaf is active at a time YaY
```

### 4th and Hardest: Using Custom Heap Class

This problem has a space of its own, build a heap data structure, or implement a priority 
queue Abstract Data Type (ADT)

Try to avoid this at any cost, in a sort problem. If interviewer asks us to implement priority ADT,
this is the backup

If we can correctly implement the heap, the rest is 4 lines of code to sort the array YaY

```python
'''
There are many definitions of heap, and most AI produces bloated Garbage Code, because that's
Production Grade and learnt from real implementation, not suitable for pressure-cooker interview
Their heap Class will have 5 methods, Yikes ! 
I wrote the below code in one sitting, zero bug. Revise this as heap implementation

We didn't write dedicated heapify function, just build the initial heap from n items
by inserting elements (push) one by one.
'''
class Heap:
    def __init__(self):
        self.heap = []
        
    
    def push(self, val):
        self.heap.append(val)

        i = len(self.heap) - 1
        # bubble up
        while i > 0:
            parent = (i - 1) // 2
            if parent >= 0 and self.heap[parent] > self.heap[i]:
                self.heap[parent], self.heap[i] = self.heap[i], self.heap[parent]
                i = parent
            else:
                break
        return
    def __bool__(self):
        return  bool(self.heap)

    def pop(self) -> int:
        if not self.heap:
            return None
        ans = self.heap[0]
        self.heap[0] = self.heap[-1]
        self.heap.pop()
        i = 0
        # bubble down
        while i < len(self.heap):
            idx = i
            left = (i*2) + 1
            right = (i*2) + 2
            if left < len(self.heap) and self.heap[left] < self.heap[idx]:
                idx = left
            if right < len(self.heap) and self.heap[right] < self.heap[idx]:
                idx = right
            if idx != i:
                self.heap[i], self.heap[idx] = self.heap[idx], self.heap[i]
                i = idx
            else:
                break
        return ans    
                

class Solution:

    def sortArray(self, nums: List[int]) -> List[int]:
        heap = Heap()
        for num in nums:
            heap.push(num)
        ans = []
        while heap:
            ans.append(heap.pop())
        return ans
# Time Complexity: O(nlogn)
# Space Complexity: O(n)
```

### Additional Stuff, If interviewer pushes **_Constant Space_** n log n sort !
- In Place sort implementation, that is hard. There is a heap version with heapify method, which constructs heap
from n items in place, but hopefully will not be needed

Below is Gemini Produced True In Place Heap Sort, this does not necessarily implement the heap class with
push/pop functionality, but uses the core bubble down logic to sort an array in place
```python
class Solution:
    def sortArray(self, nums: List[int]) -> List[int]:
        def sift_down(n: int, i: int):
            while True:
                largest = i
                left = 2 * i + 1
                right = 2 * i + 2
                if left < n and nums[left] > nums[largest]:
                    largest = left
                if right < n and nums[right] > nums[largest]:
                    largest = right
                if largest != i:
                    nums[i], nums[largest] = nums[largest], nums[i]
                    i = largest
                else:
                    break

        n = len(nums)
        # Step 1: Build Max-Heap in O(n) time bottom-up
        for i in range(n // 2 - 1, -1, -1):
            sift_down(n, i)

        # Step 2: Extract max element and move to end
        for i in range(n - 1, 0, -1):
            nums[0], nums[i] = nums[i], nums[0]  # Move current max to sorted end
            sift_down(i, 0)                      # Re-heapify reduced heap

        return nums

# Time Complexity: O(nlogn)
# Space: O(1) 
# This is the most time+space combined efficient solution possible
```

- If we have infinite memory, memory is cheap, just do counting sort, we can achieve O(n) sort time,
Just discuss this if interviewer asks theoretical situation if we can sort in linear time