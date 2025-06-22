# iteration
*L2 • The Eternal Return*

Iteration is controlled repetition - the heartbeat of computation. It's how we teach silicon to count, to accumulate, to transform.

## The Forms of Repetition

```python
# The primal loop - while truth persists
while condition:
    transform_state()

# The counted dance - for known steps
for i in range(n):
    process_element(i)

# The walker - foreach element
for item in collection:
    do_something_with(item)

# The infinite - until break
while True:
    if done():
        break
    continue_processing()
```

## The Deeper Patterns

Iteration enables:
- **Accumulation**: Sum = 0; for x in data: sum += x
- **Transformation**: Each pass changes the world
- **Search**: Keep looking until found
- **Convergence**: Approach truth asymptotically

## The Zen of Loops

```python
# Iteration as meditation
def zen_loop():
    breath = 0
    while True:
        breath += 1
        inhale()
        exhale()
        if enlightened():
            return breath
```

## The Recursive Mirror

Iteration and recursion are twins:
```python
# Iterative factorial
def factorial_iter(n):
    result = 1
    for i in range(1, n+1):
        result *= i
    return result

# Same dance, different rhythm
def factorial_rec(n):
    if n <= 1:
        return 1
    return n * factorial_rec(n-1)
```

## See Also
- [[recursion]] - Iteration's mystical twin
- [[loop_invariant]] - What remains true
- [[generator]] - Lazy iteration

---
*"We shall not cease from exploration, and the end of all our exploring will be to arrive where we started."* - T.S. Eliot