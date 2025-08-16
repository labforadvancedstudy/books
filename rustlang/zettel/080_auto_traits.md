# Auto Traits

## Core Insight
Auto traits are automatically implemented for types when all their components satisfy the trait, enabling compositional safety properties.

```rust
// Auto traits in std
auto trait Send {} // Can move between threads
auto trait Sync {} // Can share between threads
auto trait Unpin {} // Can move after pinning
auto trait UnwindSafe {} // Safe to unwind through

// Automatic implementation
struct MyStruct {
    data: String,     // Send + Sync
    count: AtomicU32, // Send + Sync
}
// MyStruct automatically implements Send + Sync

// Opting out with negative impl
impl !Send for MyStruct {}
// Now MyStruct is explicitly !Send
```

Auto trait rules:
```rust
// Struct/enum: all fields must implement
// Tuple: all elements must implement
// Array/slice: element type must implement
// Reference: depends on pointee and trait

// Special cases
&T: Send if T: Sync
&mut T: Send if T: Send
*const T: !Send + !Sync (raw pointers)
```

Creating custom auto traits (unstable):
```rust
#![feature(auto_traits)]

auto trait MyAutoTrait {}
// All types implement unless opted out
```

Auto traits enable zero-cost safety composition - properties flow through type construction.

## Connections
← [[030_marker_traits]]
← [[078_send_sync]]
→ [[153_negative_impls]]

---
Level: L5
Date: 2025-08-15
Tags: #auto-traits #markers #composition #safety