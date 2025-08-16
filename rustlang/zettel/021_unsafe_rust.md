# Unsafe Rust

## Core Insight
Unsafe blocks enable operations the compiler cannot verify, creating a trusted foundation for safe abstractions while maintaining memory safety invariants.

```rust
unsafe {
    // Five superpowers:
    1. Dereference raw pointers
    2. Call unsafe functions
    3. Access/modify mutable statics
    4. Implement unsafe traits
    5. Access union fields
}
```

Example: building safe abstraction over unsafe:
```rust
fn split_at_mut(slice: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = slice.len();
    let ptr = slice.as_mut_ptr();
    
    assert!(mid <= len);
    
    unsafe {
        (
            slice::from_raw_parts_mut(ptr, mid),
            slice::from_raw_parts_mut(ptr.add(mid), len - mid),
        )
    }
}
```

Unsafe doesn't disable borrow checker - it trusts you to uphold invariants manually. Safe APIs must encapsulate unsafe code correctly.

"Unsafe" means "I verified this is safe, trust me compiler."

## Connections
→ [[060_raw_pointers]]
→ [[061_unsafe_traits]]
→ [[062_transmute]]
→ [[063_ffi]]

---
Level: L5
Date: 2025-08-15
Tags: #unsafe #raw-pointers #memory #trust