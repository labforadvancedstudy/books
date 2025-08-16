# RAII Pattern

## Core Insight
Resource Acquisition Is Initialization ensures resources are tied to object lifetime, guaranteeing cleanup through deterministic destruction.

```rust
struct FileGuard {
    file: File,
}

impl FileGuard {
    fn new(path: &str) -> io::Result<Self> {
        Ok(FileGuard {
            file: File::open(path)?, // Resource acquired
        })
    }
}

impl Drop for FileGuard {
    fn drop(&mut self) {
        // Resource released automatically
        // Even if panic occurs
    }
}

// Usage
{
    let _guard = FileGuard::new("data.txt")?;
    // File automatically closed at scope end
} // drop() called here
```

RAII in Rust standard library:
- `Vec<T>`: memory allocation/deallocation
- `Mutex<T>`: lock acquisition/release
- `File`: file handle open/close
- Smart pointers: reference counting

No `finally` blocks needed - Drop trait handles all cleanup, even during unwinding from panics.

## Connections
← [[004_drop_trait]]
→ [[025_smart_pointers]]
→ [[069_guard_pattern]]
→ [[070_scope_guard]]

---
Level: L4
Date: 2025-08-15
Tags: #raii #resource-management #drop #patterns