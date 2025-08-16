# Future Trait

## Core Insight
Future represents an asynchronous computation that may not have finished yet, polling to completion without blocking threads.

```rust
trait Future {
    type Output;
    
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) 
        -> Poll<Self::Output>;
}

enum Poll<T> {
    Ready(T),
    Pending,
}
```

Futures are lazy and do nothing until polled. The executor drives futures by:
1. Call `poll()`
2. If `Pending`, register waker
3. When ready, waker notifies executor
4. Executor polls again

```rust
struct TimerFuture {
    deadline: Instant,
}

impl Future for TimerFuture {
    type Output = ();
    
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<()> {
        if Instant::now() >= self.deadline {
            Poll::Ready(())
        } else {
            // Register waker for later
            Poll::Pending
        }
    }
}
```

This push-based model enables efficient async I/O without thread blocking.

## Connections
← [[019_async_await]]
→ [[056_pinning]]
→ [[058_waker_api]]
→ [[059_poll_model]]

---
Level: L5
Date: 2025-08-15
Tags: #async #future #trait #polling