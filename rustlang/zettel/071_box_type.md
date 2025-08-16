# Box Type

## Core Insight
Box<T> provides heap allocation with single ownership, enabling recursive types and trait objects while maintaining move semantics.

```rust
// Heap allocation
let boxed = Box::new(5); // 5 is on heap
let stack = 5;           // 5 is on stack

// Recursive types (impossible without Box)
enum List {
    Cons(i32, Box<List>), // Box breaks infinite size
    Nil,
}

let list = List::Cons(1, 
    Box::new(List::Cons(2, 
        Box::new(List::Nil))));

// Trait objects
let drawable: Box<dyn Draw> = Box::new(Circle { radius: 5.0 });
```

Box use cases:
- Large data to avoid stack overflow
- Recursive data structures
- Trait objects (dynamic dispatch)
- Transferring ownership out of function

Performance:
```rust
// Single allocation, single pointer indirection
let boxed = Box::new([0; 1000]); // Heap allocated array

// Zero-cost conversion
let vec: Vec<i32> = Box::new([1, 2, 3]) // From box
    .into_vec(); // To vec (no reallocation)
```

Box is the simplest smart pointer - just owned heap allocation.

## Connections
← [[025_smart_pointers]]
→ [[141_box_patterns]]
→ [[142_thin_box]]

---
Level: L3
Date: 2025-08-15
Tags: #box #heap #smart-pointer #allocation