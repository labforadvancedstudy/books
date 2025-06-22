# Higher Order Functions

## Core Insight
Functions that take functions as arguments or return functions as results - they treat computation itself as a value that can be passed around and manipulated.

In elementary math, we work with numbers. In algebra, we work with variables representing numbers. In functional programming, we work with functions representing computations.

```python
def apply_twice(f, x):
    return f(f(x))

def add_one(n):
    return n + 1

result = apply_twice(add_one, 5)  # 7
```

`apply_twice` doesn't care what `f` does. It just knows to apply it twice. We've abstracted over the concept of "doing something twice."

Map, filter, reduce - the holy trinity of functional programming:

```python
numbers = [1, 2, 3, 4, 5]
squared = map(lambda x: x**2, numbers)        # Transform each
evens = filter(lambda x: x % 2 == 0, numbers) # Select some
total = reduce(lambda a, b: a + b, numbers)   # Combine all
```

These functions don't loop. They accept functions that describe what to do with elements. The looping is abstracted away.

But the real power comes from functions returning functions:

```python
def make_multiplier(n):
    def multiplier(x):
        return x * n
    return multiplier

times_two = make_multiplier(2)
times_ten = make_multiplier(10)
```

We're not just computing values. We're manufacturing new functions on demand. Each has its own captured state (the closure).

The profound realization: in languages with first-class functions, the distinction between data and code blurs. Functions are values. Values can be functions. Computation becomes something you can store, pass around, and manipulate like any other data.

## Connections
→ [[closures]]
→ [[functional_composition]]
→ [[currying]]
← [[functions_as_abstractions]]

---
Level: L5
Date: 2025-06-22
Tags: #higher-order #functions #functional-programming #abstraction