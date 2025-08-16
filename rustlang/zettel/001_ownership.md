# Ownership

## Core Insight
Every value in Rust has exactly one owner at any given time, and when the owner goes out of scope, the value is dropped.

Ownership is Rust's fundamental memory management principle. Unlike garbage-collected languages or manual memory management, Rust uses compile-time ownership rules to ensure memory safety without runtime overhead.

The three rules of ownership:
1. Each value has a variable called its owner
2. There can only be one owner at a time
3. When the owner goes out of scope, the value is dropped

```rust
let s1 = String::from("hello");
let s2 = s1; // s1 is moved to s2, s1 is no longer valid
// println!("{}", s1); // ERROR: value borrowed after move
```

This zero-cost abstraction eliminates entire classes of bugs: use-after-free, double-free, and memory leaks. The compiler enforces these rules at compile time, making runtime errors impossible.

## Connections
→ [[002_borrowing]]
→ [[003_move_semantics]]
→ [[004_drop_trait]]
→ [[005_lifetime]]

---
Level: L3
Date: 2025-08-15
Tags: #memory #safety #core #ownership