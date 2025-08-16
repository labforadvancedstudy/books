# Async Runtime

## Core Insight
Async runtimes (executors) drive futures to completion by polling them and managing the event loop, as Rust provides no built-in runtime.

```rust
// Tokio runtime
#[tokio::main] // Macro sets up runtime
async fn main() {
    let result = async_operation().await;
}

// Expands to:
fn main() {
    let runtime = tokio::runtime::Runtime::new().unwrap();
    runtime.block_on(async {
        let result = async_operation().await;
    });
}
```

Runtime responsibilities:
- **Task scheduling**: deciding which future to poll
- **Event loop**: waiting for I/O events
- **Timer management**: scheduling delayed tasks
- **Task spawning**: creating concurrent tasks

Popular runtimes:
```rust
// Tokio: feature-rich, multi-threaded
tokio::spawn(async { /* task */ });

// async-std: std-like API
async_std::task::spawn(async { /* task */ });

// smol: minimal, single-threaded
smol::spawn(async { /* task */ }).detach();
```

Runtimes are not interchangeable - futures may depend on runtime-specific features (timers, I/O).

## Connections
← [[019_async_await]]
← [[020_future_trait]]
→ [[114_executor_design]]
→ [[115_reactor_pattern]]

---
Level: L5
Date: 2025-08-15
Tags: #async #runtime #executor #event-loop