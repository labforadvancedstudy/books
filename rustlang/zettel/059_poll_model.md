# Poll Model

## Core Insight
Rust's poll-based async model inverts control - executors poll futures rather than futures blocking threads, enabling millions of concurrent tasks.

```rust
enum Poll<T> {
    Ready(T),    // Value is available
    Pending,     // Not ready, try again later
}

// Pull-based model
loop {
    match future.poll(cx) {
        Poll::Ready(value) => return value,
        Poll::Pending => {
            // Yield to executor, wait for wake
            park_thread();
        }
    }
}
```

Poll vs callback models:
```rust
// Rust: Pull-based (polling)
future.poll(cx) // Executor asks future for progress

// JavaScript: Push-based (callbacks)
promise.then(callback) // Promise calls callback when ready
```

Benefits of polling:
- Zero allocations for combinators
- Cancellation is free (just drop)
- Backpressure built-in
- No callback hell
- Composable state machines

Challenges:
- Self-referential types need pinning
- Manual state machine construction
- Complex lifetime interactions

The poll model enables Rust's zero-cost async abstractions.

## Connections
← [[020_future_trait]]
→ [[121_push_vs_pull]]
→ [[122_generator_model]]

---
Level: L5
Date: 2025-08-15
Tags: #polling #async #model #futures