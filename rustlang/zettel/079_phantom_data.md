# PhantomData

## Core Insight
PhantomData acts as a zero-sized marker to inform the compiler about type relationships without storing actual data.

```rust
use std::marker::PhantomData;

// Indicates ownership of T without storing T
struct Unique<T> {
    ptr: *const T,
    _marker: PhantomData<T>, // Zero-sized
}

// Lifetime relationship without storing reference
struct Ref<'a, T: 'a> {
    ptr: *const T,
    _marker: PhantomData<&'a T>, // Tells compiler about 'a
}

// Variance control
struct Invariant<T> {
    _marker: PhantomData<fn(T) -> T>, // Invariant over T
}
```

Common uses:
```rust
// FFI: indicate ownership
struct CString {
    ptr: *mut c_char,
    _owns: PhantomData<c_char>, // We own the pointed data
}

// Unsafe code: lifetime tracking
struct IterMut<'a, T> {
    ptr: *mut T,
    end: *mut T,
    _marker: PhantomData<&'a mut T>, // Lifetime 'a
}
```

PhantomData affects:
- Drop check
- Variance
- Auto traits (Send/Sync)
- Lifetime inference

Zero runtime cost - completely removed by compiler.

## Connections
← [[030_marker_traits]]
→ [[145_variance]]
→ [[152_drop_check]]

---
Level: L5
Date: 2025-08-15
Tags: #phantom-data #markers #zero-sized #variance