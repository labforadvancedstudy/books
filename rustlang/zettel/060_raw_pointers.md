# Raw Pointers

## Core Insight
Raw pointers (*const T and *mut T) bypass Rust's safety guarantees, enabling FFI, performance optimizations, and building safe abstractions.

```rust
let x = 5;
let raw_const: *const i32 = &x;
let mut y = 10;
let raw_mut: *mut i32 = &mut y;

// Creating raw pointers is safe
// Dereferencing requires unsafe
unsafe {
    println!("raw_const: {}", *raw_const);
    *raw_mut = 20;
}
```

Raw pointer properties:
- No ownership rules
- No lifetime tracking
- Can be null
- Can point to invalid memory
- Can alias (multiple mutable)
- No automatic cleanup

Common uses:
```rust
// Building collections
impl<T> Vec<T> {
    fn push(&mut self, value: T) {
        unsafe {
            let ptr = self.as_mut_ptr().add(self.len);
            ptr.write(value);
            self.set_len(self.len + 1);
        }
    }
}

// FFI
extern "C" {
    fn malloc(size: size_t) -> *mut c_void;
}
```

Raw pointers are the escape hatch for implementing low-level primitives.

## Connections
← [[021_unsafe_rust]]
→ [[063_ffi]]
→ [[123_pointer_arithmetic]]

---
Level: L5
Date: 2025-08-15
Tags: #raw-pointers #unsafe #memory #ffi