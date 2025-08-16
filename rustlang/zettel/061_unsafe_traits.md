# Unsafe Traits

## Core Insight
Unsafe traits require implementors to uphold invariants the compiler cannot verify, marking a contract that safe code depends on.

```rust
unsafe trait GlobalAlloc {
    unsafe fn alloc(&self, layout: Layout) -> *mut u8;
    unsafe fn dealloc(&self, ptr: *mut u8, layout: Layout);
}

// Implementing unsafe trait requires unsafe
unsafe impl GlobalAlloc for MyAllocator {
    unsafe fn alloc(&self, layout: Layout) -> *mut u8 {
        // Must uphold allocator invariants
    }
}
```

Common unsafe traits:
```rust
// Send: safe to transfer between threads
unsafe impl Send for MyType {}

// Sync: safe to share references between threads
unsafe impl Sync for MyType {}

// GlobalAlloc: memory allocator contract
// TrustedLen: iterator length exact
```

Safety contracts:
- `Send`: no thread-local state
- `Sync`: internal synchronization correct
- `GlobalAlloc`: memory alignment, no double-free
- `TrustedLen`: size_hint exact

Unsafe traits document invariants that safe abstractions rely on - violating them causes undefined behavior even in safe code.

## Connections
← [[021_unsafe_rust]]
→ [[078_send_sync]]
→ [[124_safety_contracts]]

---
Level: L5
Date: 2025-08-15
Tags: #unsafe #traits #invariants #contracts