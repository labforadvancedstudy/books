# Drop Trait

## Core Insight
The Drop trait provides deterministic destructor semantics, allowing custom cleanup code when a value goes out of scope.

```rust
struct CustomResource {
    data: String,
}

impl Drop for CustomResource {
    fn drop(&mut self) {
        println!("Cleaning up: {}", self.data);
    }
}

{
    let resource = CustomResource {
        data: String::from("important"),
    };
} // drop() called automatically here
```

Drop is called automatically when:
- Variable goes out of scope
- Ownership is transferred
- Program panics (unless `-C panic=abort`)

RAII (Resource Acquisition Is Initialization) pattern in Rust guarantees cleanup even in error cases. You cannot call `drop()` explicitly; use `std::mem::drop()` to move and drop early.

## Connections
← [[001_ownership]]
← [[003_move_semantics]]
→ [[024_raii_pattern]]
→ [[025_smart_pointers]]

---
Level: L3
Date: 2025-08-15
Tags: #memory #cleanup #trait #destructor