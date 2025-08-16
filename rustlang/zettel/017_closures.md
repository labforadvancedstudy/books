# Closures

## Core Insight
Closures are anonymous functions that capture their environment, with the compiler inferring the minimal capture mode for optimal performance.

```rust
let x = 4;
let equal_to_x = |z| z == x; // Captures x by reference

assert!(equal_to_x(4));
```

Three capture modes (compiler chooses minimal):
1. **Immutable borrow** (`Fn`): read-only access
2. **Mutable borrow** (`FnMut`): can modify captured values
3. **Move** (`FnOnce`): takes ownership

```rust
let mut count = 0;
let mut inc = || {
    count += 1; // FnMut: mutable capture
    count
};

let data = vec![1, 2, 3];
let consume = move || {
    println!("{:?}", data); // FnOnce: moved into closure
};
```

Closures are zero-cost abstractions - each has unique anonymous type, enabling inlining and optimization.

## Connections
→ [[018_iterators]]
→ [[049_fn_traits]]
→ [[050_move_keyword]]
→ [[051_closure_types]]

---
Level: L4
Date: 2025-08-15
Tags: #closures #functional #capture #anonymous-functions