# Fn Traits

## Core Insight
The Fn trait family (Fn, FnMut, FnOnce) categorizes closures by how they capture and use their environment, determining borrowing and ownership.

```rust
// FnOnce: consumes captured values, callable once
trait FnOnce<Args> {
    type Output;
    fn call_once(self, args: Args) -> Self::Output;
}

// FnMut: mutably borrows values, callable multiple times
trait FnMut<Args>: FnOnce<Args> {
    fn call_mut(&mut self, args: Args) -> Self::Output;
}

// Fn: immutably borrows values, callable multiple times
trait Fn<Args>: FnMut<Args> {
    fn call(&self, args: Args) -> Self::Output;
}
```

Closure categorization:
```rust
let s = String::from("hello");

let fn_once = move || s;           // FnOnce: moves s
let mut counter = 0;
let fn_mut = || counter += 1;      // FnMut: mutates counter
let fn_ref = || println!("{}", s); // Fn: only reads s
```

Function pointers implement all three:
```rust
fn regular_fn(x: i32) -> i32 { x * 2 }
// regular_fn implements Fn, FnMut, and FnOnce
```

The compiler infers the most permissive trait, enabling maximum flexibility.

## Connections
← [[017_closures]]
→ [[050_move_keyword]]
→ [[051_closure_types]]

---
Level: L4
Date: 2025-08-15
Tags: #closures #traits #fn-traits #capture