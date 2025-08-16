# Waker API

## Core Insight
Wakers notify executors when a future is ready to make progress, enabling efficient async I/O without busy-waiting.

```rust
struct Context<'a> {
    waker: &'a Waker,
}

impl Future for TimerFuture {
    type Output = ();
    
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<()> {
        if Instant::now() >= self.deadline {
            Poll::Ready(())
        } else {
            // Register waker to be called when timer expires
            self.timer.register_waker(cx.waker().clone());
            Poll::Pending
        }
    }
}
```

Waker lifecycle:
1. Future returns `Poll::Pending`
2. Registers waker with I/O source
3. I/O completes, calls `waker.wake()`
4. Executor re-polls future

Creating wakers:
```rust
use std::task::{RawWaker, RawWakerVTable, Waker};

unsafe fn clone_waker(data: *const ()) -> RawWaker { /* ... */ }
unsafe fn wake(data: *const ()) { /* ... */ }
unsafe fn wake_by_ref(data: *const ()) { /* ... */ }
unsafe fn drop_waker(data: *const ()) { /* ... */ }

const VTABLE: RawWakerVTable = RawWakerVTable::new(
    clone_waker, wake, wake_by_ref, drop_waker
);
```

Wakers enable futures to sleep until ready, making async efficient.

## Connections
← [[020_future_trait]]
→ [[115_reactor_pattern]]
→ [[120_wake_semantics]]

---
Level: L6
Date: 2025-08-15
Tags: #waker #async #polling #notifications