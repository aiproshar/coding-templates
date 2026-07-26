# Floyd's Cycle Detection  (Tortoise and Hare)

Two pointers, `slow` and `fast`. Slow moves one step, fast moves two steps.
If there is a cycle, fast will eventually lap slow and they collide **inside**
the cycle. If there is no cycle, fast just runs off the end and we are done.

Why must they collide? Once both pointers are inside the loop, think of it as
a circular track. Every step, fast gains exactly **one** position on slow
(2 - 1 = 1). The gap between them shrinks by one each step, so it can never
jump over slow — it will always land exactly on slow at some point. No cycle
is skippable.

The whole thing is O(N) time and O(1) space, that is the reason we love it —
no hashset, no visited array.

## Part 1: Does a cycle exist?

[LeetCode 141. Linked List Cycle](https://leetcode.com/problems/linked-list-cycle/)

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution:
    def hasCycle(self, head: Optional[ListNode]) -> bool:
        if head is None:
            return False

        slow = head
        fast = head

        while True:
            # fast moves two steps, so we must be able to take BOTH.
            # If either the next or the next.next is None, we hit the
            # end of the list -> no cycle.
            if fast.next is None or fast.next.next is None:
                return False

            slow = slow.next          # one step
            fast = fast.next.next     # two steps

            if slow is fast:          # they collided -> cycle exists
                return True
```

The only thing to be careful about is the **null checks on fast**. Fast is the
pointer that races ahead, so fast is the one that can fall off the list. Slow
never needs a null check — it is always behind fast, so if fast is safe, slow
is safe.

## Part 2: Where does the cycle begin?

Detecting a cycle is half the story. The prettier trick is finding the exact
node where the cycle starts. After the collision, do this:

- keep one pointer at the meeting point
- send a fresh pointer back to `head`
- move **both one step at a time**
- where they meet is the start of the cycle

Sounds like magic. Here is the picture — name three distances:

```
    a = head .......... to cycle start
    b = cycle start ... to meeting point
    c = meeting point . back to cycle start  (rest of the loop)

    loop length = b + c

    head ──a──► (start) ──b──► (meet)
                   ▲              │
                   └──────c───────┘
```

It works out that `a` (head to cycle start) equals `c` (meeting point back to
cycle start), give or take whole loops that just circle you back to the same
node. So one pointer starting at `head` and one starting at the meeting point,
both stepping one node at a time, arrive at the cycle start together. Just
memorize the two phases — detect the collision, then reset one pointer to
`head`.

### Code

[LeetCode 142. Linked List Cycle II](https://leetcode.com/problems/linked-list-cycle-ii/)

```python
class Solution:
    def detectCycle(self, head: Optional[ListNode]) -> Optional[ListNode]:
        if head is None:
            return None

        slow = head
        fast = head

        # Phase 1: find the meeting point (same as hasCycle)
        while True:
            if fast.next is None or fast.next.next is None:
                return None            # no cycle
            slow = slow.next
            fast = fast.next.next
            if slow is fast:
                break                  # collided, meeting point found

        # Phase 2: reset one pointer to head, walk both ONE step at a time.
        # They are guaranteed to meet at the cycle start (proof above).
        slow = head
        while slow is not fast:
            slow = slow.next
            fast = fast.next
        return slow                    # == the node where the cycle begins
```

## Part 3: Find the Duplicate Number  (same trick in disguise)

[LeetCode 287. Find the Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number/)

This one looks like an array problem, but it is Floyd's cycle detection wearing
a costume. The constraints are the tell: `n + 1` numbers, each in the range
`[1, n]`, exactly one duplicate, and we are not allowed to modify the array or
use extra space.

The trick is to treat each index as a node and `nums[i]` as its "next" pointer.
Because every value is in `[1, n]`, `nums[i]` is always a valid index — so
`i -> nums[i]` forms a linked list. Two different indices holding the **same
value** point to the same next node, which is exactly what creates a cycle,
and the **entrance to that cycle is the duplicate number**.

So this is literally "find where the cycle begins" from Part 2.

```python
class Solution:
    def findDuplicate(self, nums: List[int]) -> int:
        # Phase 1: find the meeting point.
        # slow takes one hop (i -> nums[i]), fast takes two.
        slow = 0
        fast = 0
        while True:
            slow = nums[slow]
            fast = nums[nums[fast]]     # two hops
            if slow == fast:
                break

        # Phase 2: reset slow to the start, walk both one hop at a time.
        # They meet at the cycle entrance == the duplicate value.
        slow = 0
        while slow != fast:
            slow = nums[slow]
            fast = nums[fast]
        return slow
```

Note we start both at index `0`. Index `0` can never be part of the cycle
because no value equals `0` (values are `1..n`), so `0` is guaranteed to be
outside the loop, sitting on the "tail" — the perfect `head` for the algorithm.

### What Floyd returns: the entrance, not the node that closes the loop

Walk the example `nums = [1, 3, 4, 2, 2]`. Following `i -> nums[i]` from index
`0` gives the chain of visited nodes:

```
    0 → 1 → 3 → 2 → 4 → 2 → 4 → …      (first value to repeat is 2)
```

Two indices hold the value `2` (`nums[3] = 2` and `nums[4] = 2`), so two arrows
point into node `2`. Two arrows in = that node is the loop entrance:

```
    tail          loop
    ────────   ┌────────┐
    0 → 1 → 3 → 2 → 4 ──┘
                ▲
             entrance = duplicate = 2
```

The `──┘` means `4`'s pointer curves back to `2` (not onward to a new node), so
you circle `2 → 4 → 2 → 4 …` forever.

Here is the part people trip on: **Floyd returns the node where the loop begins
— the entrance you land on — not the node whose pointer closes the loop.** In
the picture `4` is the node that *causes* the cycle (its arrow reaches back),
but Floyd does not return `4`. It returns `2`, the node that `4` points to.

That is exactly why phase 2 ends with `return slow`. After the second walk
`slow` sits on the entrance (`2`), which is the duplicate. `nums[slow]` would
step one node too far onto `4` (the causing node) — that is the classic bug.

## Time and Space Complexity

- **Time: O(N)** — Phase 1 walks at most a couple passes over the structure
  before colliding, Phase 2 walks `a` steps. Both linear.
- **Space: O(1)** — just two pointers. This is the whole point of Floyd over a
  hashset / visited-array approach, which would cost O(N) space.

## MUST DO while revisit problems

* https://leetcode.com/problems/linked-list-cycle/  (does a cycle exist)

* https://leetcode.com/problems/linked-list-cycle-ii/  (where does it begin)

* https://leetcode.com/problems/find-the-duplicate-number/  (cycle begin in disguise)

* https://leetcode.com/problems/happy-number/  (cycle detection on a number sequence)

Note: a hashset of visited nodes also solves all of these in O(N) time — but at
O(N) space. Reach for Floyd when the interviewer says "constant space".
