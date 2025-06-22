# Closures

## Core Insight
Closures are functions that remember their birthplace - they capture and carry their surrounding context wherever they go, like computational nostalgia.

A function normally forgets where it came from. Call it from anywhere, it behaves the same. But closures remember:

```python
def make_counter():
    count = 0
    def increment():
        nonlocal count
        count += 1
        return count
    return increment

counter1 = make_counter()
counter2 = make_counter()

print(counter1())  # 1
print(counter1())  # 2
print(counter2())  # 1 - different counter!
```

Each call to `make_counter` creates a new universe with its own `count`. The `increment` function born in that universe remembers it forever. Even after `make_counter` returns, its local variables live on, captured by the closure.

This is almost magical. In languages without closures, local variables die when functions return. But closures grant them immortality.

Why are they called "closures"? Because they "close over" free variables:

```javascript
function makeAdder(x) {
    return function(y) {
        return x + y;  // x is "free" in this function
    };
}

const add5 = makeAdder(5);
// The function "closes over" x, binding it to 5
```

Closures enable:
- Private state (the captured variables)
- Function factories (make specialized functions)
- Callbacks that remember context
- The entire paradigm of functional programming

The profound insight: closures break the tyranny of stack-based execution. Variables don't have to die when their stack frame pops. Functions aren't just code - they're code plus the environment where they were born.

## Connections
→ [[lexical_scope]]
→ [[function_environments]]
→ [[garbage_collection]]
← [[higher_order_functions]]

---
Level: L5
Date: 2025-06-22
Tags: #closures #scope #functions #memory