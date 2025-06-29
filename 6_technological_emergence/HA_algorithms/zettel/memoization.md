# Memoization (L6)

## The Art of Computational Memory

Memoization is an optimization technique where we store the results of expensive function calls and return the cached result when the same inputs occur again. It's the algorithmic equivalent of learning from experience.

## The Core Insight

Why solve the same problem twice? This simple question drives memoization. By remembering past solutions, we trade space for time, transforming exponential problems into polynomial ones.

## How It Works

```python
# Without memoization - exponential time
def fib_naive(n):
    if n <= 1:
        return n
    return fib_naive(n-1) + fib_naive(n-2)

# With memoization - linear time
def memoize(func):
    cache = {}
    def wrapper(*args):
        if args not in cache:
            cache[args] = func(*args)
        return cache[args]
    return wrapper

@memoize
def fib_fast(n):
    if n <= 1:
        return n
    return fib_fast(n-1) + fib_fast(n-2)
```

## The Philosophical Depth

Memoization embodies a profound truth: computation is often redundant. Nature doesn't recalculate the laws of physics for each falling apple. Similarly, memoization recognizes that many problems contain overlapping subproblems.

## Real-World Analogies

- **Learning**: You don't re-derive math theorems each time you use them
- **Cooking**: Meal prep stores ready-made components
- **Navigation**: Remembering routes instead of exploring each time
- **Language**: Caching word meanings instead of parsing from scratch

## When to Memoize

Memoization shines when:
1. Function is pure (same input → same output)
2. Computation is expensive
3. Same inputs occur frequently
4. Subproblems overlap significantly

## Advanced Patterns

```python
# LRU (Least Recently Used) memoization
from functools import lru_cache

@lru_cache(maxsize=128)
def expensive_operation(x, y):
    # Complex computation
    return result

# Manual cache management
class Memoizer:
    def __init__(self):
        self.cache = {}
    
    def compute(self, key, func):
        if key not in self.cache:
            self.cache[key] = func()
        return self.cache[key]
    
    def clear(self):
        self.cache.clear()
```

## The Trade-offs

- **Memory vs Time**: Cache growth can become problematic
- **Complexity**: Adds layer of indirection
- **Cache Invalidation**: When do stored results become stale?
- **Parallelism**: Cache access needs synchronization

## Memoization in Nature

The concept appears throughout reality:
- **DNA**: Storing genetic solutions
- **Memory**: Brain caching experiences
- **Culture**: Societies remembering solutions
- **Evolution**: Species "memoizing" adaptations

## Beyond Simple Caching

Memoization connects to deeper concepts:
- **Dynamic Programming**: Systematic memoization
- **Machine Learning**: Memoizing patterns in data
- **Database Indexing**: Memoizing query results
- **CDN**: Memoizing web content globally

## The Existential Question

If we can memoize computation, what does this say about consciousness? Are we just biological memoization machines, caching responses to stimuli?

## Common Pitfalls

- **Memory Leaks**: Unbounded cache growth
- **Stale Data**: Cached results that should expire
- **Side Effects**: Memoizing impure functions
- **Over-memoization**: Caching trivial computations

## Connection to Other Concepts

- **Dynamic Programming** (L5): Structured memoization
- **Optimization** (L4): Memoization as optimization technique
- **Recursion** (L4): Often combined with memoization
- **Distributed Computing** (L7): Distributed memoization challenges