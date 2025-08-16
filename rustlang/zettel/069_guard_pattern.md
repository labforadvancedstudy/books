# Guard Pattern

## Core Insight
Guards use RAII to ensure cleanup happens automatically when leaving scope, making resource management exception-safe and ergonomic.

```rust
// MutexGuard automatically unlocks
let guard = mutex.lock().unwrap();
*guard += 1;
// Mutex unlocked when guard drops

// Custom guard implementation
struct FileGuard {
    file: File,
}

impl Drop for FileGuard {
    fn drop(&mut self) {
        self.file.sync_all().unwrap();
        println!("File synced and closed");
    }
}

// Scope guard pattern
fn with_cleanup<F: FnOnce()>(f: F) {
    struct Guard<F: FnOnce()>(Option<F>);
    
    impl<F: FnOnce()> Drop for Guard<F> {
        fn drop(&mut self) {
            if let Some(f) = self.0.take() {
                f();
            }
        }
    }
    
    let _guard = Guard(Some(f));
    // Do work...
} // Cleanup runs here
```

Common guards in std:
- `MutexGuard`: unlocks mutex
- `RwLockReadGuard/WriteGuard`: unlocks RwLock
- `RefMut/Ref`: RefCell borrow tracking

Guards make "undo on error" automatic and unforgettable.

## Connections
← [[024_raii_pattern]]
← [[067_mutex]]
→ [[070_scope_guard]]

---
Level: L4
Date: 2025-08-15
Tags: #guard #raii #patterns #resource-management