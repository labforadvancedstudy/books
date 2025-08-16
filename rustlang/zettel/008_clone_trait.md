# Clone Trait

## Core Insight
Clone provides explicit deep copying with `.clone()`, making expensive operations visible in code.

```rust
#[derive(Clone)]
struct Buffer {
    data: Vec<u8>,
}

let buf1 = Buffer { data: vec![1, 2, 3] };
let buf2 = buf1.clone(); // Explicit deep copy
// Both buf1 and buf2 are valid and independent
```

Clone vs Copy:
- Clone: explicit, can be expensive, heap allocation allowed
- Copy: implicit, must be cheap, stack only

```rust
let s1 = String::from("hello");
let s2 = s1.clone(); // Allocates new heap memory
let n1 = 42;
let n2 = n1; // Copy: no clone() needed
```

Making clones explicit prevents accidental performance issues. The compiler never inserts clone() calls automatically.

## Connections
← [[003_move_semantics]]
← [[007_copy_trait]]
→ [[031_cow_type]]

---
Level: L3
Date: 2025-08-15
Tags: #trait #clone #deep-copy #performance