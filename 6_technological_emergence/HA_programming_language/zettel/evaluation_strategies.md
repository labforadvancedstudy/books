# Evaluation Strategies

## Core Insight
Evaluation strategy is the heartbeat of computation - it determines when and how expressions come alive, transforming from potential to actual.

Consider this innocent-looking code:
```python
result = expensive_calculation(x) or cheap_calculation(y)
```

When does `expensive_calculation` run? When does `cheap_calculation` run? The answer reveals the soul of the language.

**Eager evaluation** says: "Compute everything now, ask questions later." Call both functions, then check the 'or'. Simple, predictable, potentially wasteful.

**Lazy evaluation** says: "Compute only when needed." If `expensive_calculation` returns true, never call `cheap_calculation`. Smart, efficient, potentially confusing.

But it goes deeper. Consider:
```haskell
ones = 1 : ones  -- Infinite list of 1s in Haskell
take 5 ones      -- [1,1,1,1,1]
```

In an eager language, defining `ones` would hang forever. In lazy Haskell, it's fine - compute only what's needed. The same syntax means radically different things.

Call-by-value: evaluate arguments before function call.
Call-by-name: pass expressions, evaluate when used.
Call-by-need: call-by-name + memoization.

Each strategy creates a different computational universe. Eager languages live in a world of values. Lazy languages live in a world of promises. The strategy shapes everything: performance, debugging, even what programs are possible to write.

## Connections
→ [[lazy_evaluation]]
→ [[strict_evaluation]]
→ [[thunks_and_promises]]
← [[functions_as_abstractions]]

---
Level: L5
Date: 2025-06-22
Tags: #evaluation #strategies #lazy #eager #semantics