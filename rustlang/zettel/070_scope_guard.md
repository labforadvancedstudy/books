# Scope Guard

## Core Insight
Scope guards execute arbitrary code when leaving scope, ensuring cleanup happens even during panics or early returns.

```rust
// Basic scope guard
macro_rules! defer {
    ($e:expr) => {
        struct Guard<F: FnOnce()>(Option<F>);
        impl<F: FnOnce()> Drop for Guard<F> {
            fn drop(&mut self) {
                self.0.take().map(|f| f());
            }
        }
        let _guard = Guard(Some(|| $e));
    };
}

fn process_file() -> Result<(), Error> {
    let temp = create_temp_file()?;
    defer!(remove_temp_file(&temp)); // Always runs
    
    risky_operation()?; // Even if this fails
    another_operation()?; // Or this
    Ok(())
} // Temp file removed here

// Conditional execution
struct ScopeGuard<F: FnMut()> {
    f: Option<F>,
    run_on_drop: bool,
}

impl<F: FnMut()> ScopeGuard<F> {
    fn dismiss(mut self) {
        self.run_on_drop = false;
    }
}
```

Use cases:
- Temporary file cleanup
- Transaction rollback
- Lock release
- State restoration

Scope guards provide Go-like `defer` semantics in Rust.

## Connections
← [[069_guard_pattern]]
← [[024_raii_pattern]]
→ [[140_defer_pattern]]

---
Level: L4
Date: 2025-08-15
Tags: #scope-guard #cleanup #defer #patterns