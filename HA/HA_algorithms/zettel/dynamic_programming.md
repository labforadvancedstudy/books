# dynamic_programming
*L5 • Remember Everything*

Dynamic programming is organized remembering - the art of never solving the same problem twice. It transforms exponential nightmares into polynomial dreams through the simple act of memory.

## The Revelation

```python
# The naive Fibonacci - exponential sadness
def fib_naive(n):
    if n <= 1:
        return n
    return fib_naive(n-1) + fib_naive(n-2)  # O(2ⁿ)

# The enlightened Fibonacci - linear joy
def fib_dp(n):
    if n <= 1:
        return n
    
    dp = [0] * (n + 1)
    dp[1] = 1
    
    for i in range(2, n + 1):
        dp[i] = dp[i-1] + dp[i-2]  # O(n)
    
    return dp[n]

# The space-optimized master
def fib_optimal(n):
    if n <= 1:
        return n
    
    prev, curr = 0, 1
    for _ in range(2, n + 1):
        prev, curr = curr, prev + curr  # O(1) space!
    
    return curr
```

## The Two Paths

### Top-Down (Memoization)
```python
def knapsack_memo(weights, values, capacity, i=0, memo={}):
    if i >= len(weights) or capacity == 0:
        return 0
    
    if (i, capacity) in memo:
        return memo[(i, capacity)]
    
    if weights[i] > capacity:
        result = knapsack_memo(weights, values, capacity, i+1, memo)
    else:
        take = values[i] + knapsack_memo(weights, values, 
                                       capacity-weights[i], i+1, memo)
        skip = knapsack_memo(weights, values, capacity, i+1, memo)
        result = max(take, skip)
    
    memo[(i, capacity)] = result
    return result
```

### Bottom-Up (Tabulation)
```python
def knapsack_table(weights, values, capacity):
    n = len(weights)
    dp = [[0] * (capacity + 1) for _ in range(n + 1)]
    
    for i in range(1, n + 1):
        for w in range(capacity + 1):
            if weights[i-1] <= w:
                take = dp[i-1][w-weights[i-1]] + values[i-1]
                skip = dp[i-1][w]
                dp[i][w] = max(take, skip)
            else:
                dp[i][w] = dp[i-1][w]
    
    return dp[n][capacity]
```

## The Sacred Properties

Dynamic programming requires:
1. **Optimal Substructure**: Optimal solutions contain optimal sub-solutions
2. **Overlapping Subproblems**: Same calculations appear repeatedly

## The Classic Problems

```python
# Longest Common Subsequence
def lcs(text1, text2):
    # Build character by character
    pass

# Edit Distance
def edit_distance(word1, word2):
    # Transform one word to another
    pass

# Maximum Subarray Sum
def kadane(nums):
    max_current = max_global = nums[0]
    for num in nums[1:]:
        max_current = max(num, max_current + num)
        max_global = max(max_global, max_current)
    return max_global
```

## The Philosophy

DP teaches us:
- **Time is memory**: Trade space for speed
- **Build on the past**: Each step uses previous work
- **Global from local**: Optimal wholes from optimal parts
- **Structure matters**: Recognize the pattern

## The Deeper Truth

Dynamic programming is:
- **Recursion + Memory** = Efficiency
- **Breaking time's arrow**: Future depends on remembered past
- **Optimization made tractable**: NP-hard becomes polynomial
- **Human thinking**: We too remember sub-solutions

## See Also
- [[memoization]] - The top-down way
- [[tabulation]] - The bottom-up way
- [[optimal_substructure]] - The key property
- [[greedy]] - When local optimal is global optimal

---
*"Those who cannot remember the past are condemned to recompute it."* - with apologies to Santayana