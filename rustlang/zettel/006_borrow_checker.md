# Borrow Checker

## Core Insight
The borrow checker is Rust's compile-time system that tracks ownership and borrowing rules, eliminating memory safety bugs before runtime.

The borrow checker enforces:
1. One mutable reference XOR many immutable references
2. References must always be valid
3. No reference outlives its data

```rust
let mut data = vec![1, 2, 3];
let r1 = &data[0];     // immutable borrow
// data.push(4);        // ERROR: cannot mutate while borrowed
println!("{}", r1);     // last use of r1
data.push(4);           // OK: r1 no longer in use (NLL)
```

Non-Lexical Lifetimes (NLL) makes the checker smarter - borrows end at last use, not scope end. The checker runs as part of compilation, with zero runtime overhead.

Complex cases may require restructuring code to satisfy the checker, but this forces cleaner architecture and eliminates entire bug classes.

## Connections
← [[002_borrowing]]
← [[005_lifetime]]
→ [[028_nll]]
→ [[029_polonius]]

---
Level: L4
Date: 2025-08-15
Tags: #compiler #safety #borrowing #static-analysis