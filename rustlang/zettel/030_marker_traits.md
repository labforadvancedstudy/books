# Marker Traits

## Core Insight
Marker traits have no methods but mark types with properties the compiler uses for safety and optimization decisions.

Key marker traits:
```rust
// Types that can be sent between threads
pub unsafe auto trait Send {}

// Types whose references can be sent between threads
pub unsafe auto trait Sync {}

// Types that can be copied bitwise
pub trait Copy: Clone {}

// Types with predictable memory layout
pub trait Sized {} // All types except [T] and dyn Trait

// Types that can be safely zeroed
pub unsafe trait Unpin {}
```

Auto traits are implemented automatically unless explicitly opted out:
```rust
struct MyType {
    data: Vec<u8>, // Vec is Send + Sync
} // MyType is automatically Send + Sync

struct NotSend {
    ptr: *const u8, // Raw pointers are !Send
} // NotSend is automatically !Send
```

Negative impl to opt-out:
```rust
impl !Send for NotSend {}
```

Marker traits encode type system properties that affect memory safety and thread safety.

## Connections
← [[007_copy_trait]]
→ [[078_send_sync]]
→ [[079_phantom_data]]
→ [[080_auto_traits]]

---
Level: L5
Date: 2025-08-15
Tags: #traits #markers #type-system #safety