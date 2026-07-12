# Dynamic Programming Framework

Framework from Andrey Grehov's [Dynamic Programming for Beginners](https://github.com/andreygrehov/dp) course
(YouTube course, solutions in Go, lecture3-lecture17)

DP is both easy and hard, if you cannot figure things out, for example the state space tree
and the base cases, you will circle around and produce buddy code, which is impossible to debug

But when everything clicks, its usually 3-4 lines of code and we done

Always try to start from top-down recursive approach, we will find similarities to backtrack + dfs
Memoize so our Time Complexity reduces from state space C^N to O(N)

## The Framework
- ### Identify Objective Function
- ### Identify Base Cases
- ### Recurrence Relation
      Visualize the State Space Tree, thats the most important thing
- ### Order of Computation
        Usually Top Down for Recursive (Easier but stack overflow risk)
        Bottom Up for Iterative (HARD)
- ### Where is the Answer
        dp[n] usually for bottom up, topdown we design Objective function in a way it returns
        There are also exceptions, we can have external global state ect

## Framework in Action: Climbing Stairs

[LeetCode 70. Climbing Stairs](https://leetcode.com/problems/climbing-stairs/)

### Top-Down Recursive (Memoized)

```python
class Solution:
    @cache
    def climbStairs(self, n: int) -> int:
        """
        Framework:
            Step 1: Objective Function: f(i) = number of distinct ways to reach step i
            Step 2: Base Cases: f(0) = 1, f(1) = 1
            Step 3: Recurrence Relation: f(n) = f(n-1) + f(n-2)
            Step 4: Order of Computation: top-down, @cache memoizes so 2^N -> O(N)
            Step 5: Where is the Answer: the return value of climbStairs(n)
        """
        if n == 0 or n == 1:
            return 1
        return self.climbStairs(n-1) + self.climbStairs(n-2)
```

### Bottom-Up Iterative

```python
class Solution:
    def climbStairs(self, n: int) -> int:
        """
        Framework:
            Step 1: Objective Function: dp[i] = number of distinct ways to reach step i
            Step 2: Base Cases: dp[0] = 1, dp[1] = 1
            Step 3: Recurrence Relation: dp[i] = dp[i-1] + dp[i-2]
            Step 4: Order of Computation: bottom-up, smaller subproblems first -> O(N)
            Step 5: Where is the Answer: dp[n]
        """
        dp = [0] * (n+1)
        dp[0] = 1
        dp[1] = 1
        for i in range(2, n+1):
            dp[i] = dp[i-1] + dp[i-2]
        return dp[n]
```

## Framework in Action: Min Cost Climbing Stairs (Optimization)

[LeetCode 746. Min Cost Climbing Stairs](https://leetcode.com/problems/min-cost-climbing-stairs/)

Same staircase, but now an optimization problem instead of counting:
counting problems use `+` in the recurrence, optimization problems use `min`/`max`

### Top-Down Recursive (Memoized)

```python
class Solution:
    def minCostClimbingStairs(self, cost: List[int]) -> int:
        """
        Framework:
            Step 1: Objective Function: f(i) = min cost to reach step i
            Step 2: Base Cases: f(0) = 0, f(1) = 0 (can start at step 0 or 1 for free)
            Step 3: Recurrence Relation: f(i) = min(f(i-1) + cost[i-1], f(i-2) + cost[i-2])
            Step 4: Order of Computation: top-down, @cache memoizes so 2^N -> O(N)
            Step 5: Where is the Answer: f(n), the top is one past the last step
        """
        @cache
        def f(i: int) -> int:
            if i == 0 or i == 1:
                return 0
            return min(f(i-1) + cost[i-1], f(i-2) + cost[i-2])
        return f(len(cost))
```

### Bottom-Up Iterative

```python
class Solution:
    def minCostClimbingStairs(self, cost: List[int]) -> int:
        """
        Framework:
            Step 1: Objective Function: dp[i] = min cost to reach step i
            Step 2: Base Cases: dp[0] = 0, dp[1] = 0 (can start at step 0 or 1 for free)
            Step 3: Recurrence Relation: dp[i] = min(dp[i-1] + cost[i-1], dp[i-2] + cost[i-2])
            Step 4: Order of Computation: bottom-up, smaller subproblems first -> O(N)
            Step 5: Where is the Answer: dp[n], the top is one past the last step
        """
        n = len(cost)
        dp = [0] * (n+1)
        dp[0] = 0
        dp[1] = 0
        for i in range(2, n+1):
            dp[i] = min(dp[i-1] + cost[i-1], dp[i-2] + cost[i-2])
        return dp[n]
```

