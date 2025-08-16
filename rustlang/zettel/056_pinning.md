# Pinning

## Core Insight
Pin guarantees a value won't move in memory, enabling self-referential structures required by async state machines.

```rust
use std::pin::Pin;

// Self-referential struct (normally impossible)
struct SelfRef {
    data: String,
    ptr: *const String, // Points to data field
}

// Pin prevents moving after references created
let mut pinned = Box::pin(SelfRef::new());
// pinned.data cannot move now
```

Why async needs pinning:
```rust
async fn example() {
    let data = vec![1, 2, 3];
    let ref_to_data = &data; // Reference into async state
    some_async_op().await;   // Yield point
    use_ref(ref_to_data);    // Reference must still be valid
}
// Compiler generates self-referential state machine
```

Pin API:
```rust
impl Future for MyFuture {
    fn poll(self: Pin<&mut Self>, cx: &mut Context) -> Poll<Output> {
        // self is pinned, safe to create internal references
    }
}
```

Unpin marker trait:
- Most types are Unpin (can move even when pinned)
- !Unpin types truly can't move when pinned
- Futures from async fn are !Unpin

## Connections
← [[020_future_trait]]
→ [[116_unpin_trait]]
→ [[117_pin_project]]

---
Level: L6
Date: 2025-08-15
Tags: #pinning #async #memory #self-referential