# Async Traits

## Core Insight
Async trait methods are challenging because they return opaque types, requiring workarounds until native support lands.

Current limitations:
```rust
// This doesn't work (yet)
trait AsyncTrait {
    async fn method(&self) -> Result<String, Error>;
    // Error: async fn in traits not supported
}
```

Workaround with async-trait macro:
```rust
use async_trait::async_trait;

#[async_trait]
trait AsyncTrait {
    async fn method(&self) -> Result<String, Error>;
}

#[async_trait]
impl AsyncTrait for MyType {
    async fn method(&self) -> Result<String, Error> {
        // Macro converts to Box<dyn Future>
    }
}
```

Manual desugaring:
```rust
trait AsyncTrait {
    fn method(&self) -> impl Future<Output = Result<String, Error>>;
    // Also doesn't work in traits (yet)
}

// Must use trait objects
trait AsyncTrait {
    fn method(&self) -> Pin<Box<dyn Future<Output = Result<String, Error>>>>;
}
```

Future solution (in progress):
- Async fn in traits (stabilizing)
- Return position impl trait (RPIT)
- Type alias impl trait (TAIT)

## Connections
← [[019_async_await]]
→ [[118_rpit]]
→ [[119_tait]]

---
Level: L5
Date: 2025-08-15
Tags: #async #traits #futures #workarounds