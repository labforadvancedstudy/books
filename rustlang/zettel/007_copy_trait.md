# Copy Trait

## Core Insight
Types implementing Copy are duplicated instead of moved, making them behave like primitive values with stack-only semantics.

```rust
#[derive(Copy, Clone)]
struct Point {
    x: i32,
    y: i32,
}

let p1 = Point { x: 1, y: 2 };
let p2 = p1; // p1 is copied, not moved
println!("{}, {}", p1.x, p1.y); // p1 still valid!
```

Copy requirements:
- All fields must be Copy
- Cannot implement Drop
- Must be cheap to copy (stack only)
- Must implement Clone

Built-in Copy types:
- All integers and floats
- bool and char
- Tuples of Copy types
- Arrays of Copy types
- Raw pointers

Copy makes code more ergonomic for small, simple types while maintaining explicit costs for heap-allocated data.

## Connections
← [[003_move_semantics]]
→ [[008_clone_trait]]
→ [[030_marker_traits]]

---
Level: L3
Date: 2025-08-15
Tags: #trait #copy #stack #semantics