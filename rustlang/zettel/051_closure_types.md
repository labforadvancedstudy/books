# Closure Types

## Core Insight
Each closure has a unique, anonymous type that implements the appropriate Fn traits, enabling zero-cost abstractions through monomorphization.

```rust
let add_one = |x: i32| x + 1;
let add_two = |x: i32| x + 2;

// These have DIFFERENT types, even with same signature
// type_of(add_one) != type_of(add_two)

// Can't do this:
// let mut closure = add_one;
// closure = add_two; // ERROR: type mismatch
```

Closure type anatomy:
```rust
// Conceptually, compiler generates:
struct ClosureType<'a> {
    captured_var: &'a i32, // Captured environment
}

impl<'a> Fn(i32) -> i32 for ClosureType<'a> {
    fn call(&self, x: i32) -> i32 {
        x + self.captured_var
    }
}
```

Using closures generically:
```rust
// Each call gets monomorphized
fn apply<F: Fn(i32) -> i32>(f: F, x: i32) -> i32 {
    f(x)
}

// Dynamic dispatch with trait objects
let closures: Vec<Box<dyn Fn(i32) -> i32>> = vec![
    Box::new(|x| x + 1),
    Box::new(|x| x * 2),
];
```

Unique types enable aggressive optimization - each closure call site can be inlined.

## Connections
← [[017_closures]]
← [[049_fn_traits]]
→ [[109_closure_size]]

---
Level: L5
Date: 2025-08-15
Tags: #closures #types #monomorphization #optimization