# Zero Cost Abstractions

## Core Insight
You don't pay for what you don't use, and what you do use couldn't be hand-coded better - abstractions compile away completely.

Examples of zero-cost:
```rust
// Iterator chains compile to same assembly as manual loops
let sum: i32 = numbers.iter()
    .filter(|&&x| x > 0)
    .map(|&x| x * 2)
    .sum();

// Compiles to equivalent of:
let mut sum = 0;
for &x in numbers {
    if x > 0 {
        sum += x * 2;
    }
}
```

How Rust achieves zero-cost:
- **Monomorphization**: generics generate specialized code
- **Inlining**: small functions disappear
- **LLVM optimization**: aggressive optimization passes
- **No runtime**: no GC, no reflection, no virtual calls (unless `dyn`)

```rust
// Option<T> has same size as T for non-nullable pointers
size_of::<Option<&T>>() == size_of::<&T>()
```

The abstraction penalty is paid at compile time, not runtime.

## Connections
→ [[010_generic_types]]
→ [[018_iterators]]
→ [[064_monomorphization]]
→ [[065_llvm_optimization]]

---
Level: L6
Date: 2025-08-15
Tags: #performance #zero-cost #abstractions #optimization