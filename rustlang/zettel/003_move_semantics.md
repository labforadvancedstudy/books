# Move Semantics

## Core Insight
Move semantics transfer ownership of heap-allocated data without copying, making resource management explicit and efficient.

When you assign or pass a non-Copy type, Rust moves ownership:

```rust
let vec1 = vec![1, 2, 3];
let vec2 = vec1; // vec1 moved to vec2
// vec1 is now invalid

fn take_ownership(v: Vec<i32>) {
    // v owns the vector
} // v goes out of scope and vector is dropped

let vec3 = vec![4, 5, 6];
take_ownership(vec3);
// vec3 is no longer valid here
```

Types implementing `Copy` trait (like integers) are copied instead of moved. This distinction makes heap allocation costs explicit in the type system.

Move semantics prevent:
- Double-free errors
- Use-after-move bugs
- Implicit expensive copies

## Connections
← [[001_ownership]]
→ [[007_copy_trait]]
→ [[008_clone_trait]]
→ [[004_drop_trait]]

---
Level: L3
Date: 2025-08-15
Tags: #memory #move #semantics #ownership