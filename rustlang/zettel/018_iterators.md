# Iterators

## Core Insight
Iterators provide lazy, composable sequences with zero-cost abstractions - iterator chains compile to the same assembly as hand-written loops.

```rust
trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
    // 70+ provided methods...
}

let sum: u32 = vec![1, 2, 3, 4, 5]
    .iter()
    .filter(|&&x| x % 2 == 0)
    .map(|x| x * x)
    .sum(); // 4 + 16 = 20
```

Iterator adaptors are lazy - no work until consumed:
```rust
let iter = (0..).filter(|x| x % 2 == 0); // Infinite, lazy
let first_five: Vec<_> = iter.take(5).collect(); // [0, 2, 4, 6, 8]
```

Three iteration methods:
- `iter()`: borrows items (`&T`)
- `iter_mut()`: mutable borrows (`&mut T`)
- `into_iter()`: takes ownership (`T`)

The compiler optimizes iterator chains into efficient loops through inlining and LLVM optimization.

## Connections
← [[017_closures]]
→ [[052_iterator_trait]]
→ [[053_lazy_evaluation]]
→ [[054_collect_method]]

---
Level: L4
Date: 2025-08-15
Tags: #iterators #lazy #functional #zero-cost