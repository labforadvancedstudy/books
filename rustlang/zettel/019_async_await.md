# Async/Await

## Core Insight
Async/await transforms asynchronous code into state machines, enabling millions of concurrent tasks without OS threads.

```rust
async fn fetch_data(url: &str) -> Result<String, Error> {
    let response = reqwest::get(url).await?;
    let body = response.text().await?;
    Ok(body)
}

// Desugars to state machine implementing Future trait
```

Key concepts:
- `async fn` returns `impl Future<Output = T>`
- `.await` yields control to executor
- No threading - pure cooperative multitasking
- Zero-cost: compiles to state machines

```rust
// Concurrent execution
let (a, b, c) = tokio::join!(
    fetch_data("url1"),
    fetch_data("url2"),
    fetch_data("url3")
);
```

Executors (tokio, async-std) drive futures to completion. Tasks are scheduled on thread pool, not OS threads - enabling millions of concurrent operations.

## Connections
→ [[020_future_trait]]
→ [[055_async_runtime]]
→ [[056_pinning]]
→ [[057_async_traits]]

---
Level: L5
Date: 2025-08-15
Tags: #async #concurrency #futures #state-machines