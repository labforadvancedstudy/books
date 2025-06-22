# big_o
*L4 • The Lens of Infinity*

Big O notation is how we see algorithms at the limit - stripping away constants and focusing on growth. It's the telescope through which we view computational complexity.

## The Hierarchy of Growth

```python
# O(1) - Constant - The speed of light
def get_first(array):
    return array[0]

# O(log n) - Logarithmic - Divine efficiency  
def binary_search(array, target):
    left, right = 0, len(array) - 1
    while left <= right:
        mid = (left + right) // 2
        if array[mid] == target:
            return mid
        elif array[mid] < target:
            left = mid + 1
        else:
            right = mid - 1

# O(n) - Linear - The honest walk
def sum_array(array):
    return sum(array)

# O(n log n) - The sweet spot of sorting
def merge_sort(array):
    # Divide and conquer elegance
    pass

# O(n²) - Quadratic - The nested burden
def bubble_sort(array):
    for i in range(len(array)):
        for j in range(len(array)-1):
            if array[j] > array[j+1]:
                array[j], array[j+1] = array[j+1], array[j]

# O(2ⁿ) - Exponential - The combinatorial explosion
def subsets(array):
    if not array:
        return [[]]
    first, *rest = array
    rest_subsets = subsets(rest)
    return rest_subsets + [[first] + s for s in rest_subsets]

# O(n!) - Factorial - The traveling salesman's nightmare
def permutations(array):
    # All possible orderings
    pass
```

## The Zen of Asymptotes

Big O teaches us:
- **Constants don't matter** at infinity
- **Lower order terms** vanish in the limit
- **Worst case** prepares us for reality
- **Growth rate** determines feasibility

## The Hidden Dimensions

```python
# Space complexity matters too
def memory_hog(n):
    return [[0] * n for _ in range(n)]  # O(n²) space

# Sometimes we trade space for time
def memoized_fib(n, cache={}):
    if n in cache:
        return cache[n]  # O(1) lookup
    if n <= 1:
        return n
    cache[n] = memoized_fib(n-1) + memoized_fib(n-2)
    return cache[n]  # O(n) time, O(n) space
```

## The Philosophical Weight

Big O reveals:
- **Some problems are fundamentally harder**
- **Efficiency has theoretical limits**
- **P vs NP** - The million dollar question
- **Approximation** may be our only hope

## The Practical Wisdom

```python
# O(n) can be slower than O(n²) for small n
def reality_check(n):
    if n < 1000:
        return simple_but_quadratic(n)
    else:
        return complex_but_linear(n)
```

## See Also
- [[complexity_classes]] - P, NP, and beyond
- [[amortized_analysis]] - Average case magic
- [[space_complexity]] - The other dimension
- [[P_vs_NP]] - The ultimate question

---
*"In the limit, all that matters is how we grow."*