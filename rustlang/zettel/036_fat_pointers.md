# Fat Pointers

## Core Insight
Fat pointers carry metadata alongside the data pointer, enabling dynamic sizing and dispatch without separate allocations.

Types of fat pointers (2x pointer size):

```rust
// Slice: pointer + length
let slice: &[i32] = &[1, 2, 3];
// Internal: (*const i32, usize)

// String slice: pointer + length  
let s: &str = "hello";
// Internal: (*const u8, usize)

// Trait object: pointer + vtable
let obj: &dyn Display = &42;
// Internal: (*const i32, *const DisplayVTable)
```

Memory layout:
```rust
size_of::<&i32>() == 8         // Thin pointer
size_of::<&[i32]>() == 16      // Fat pointer (ptr + len)
size_of::<&dyn Trait>() == 16  // Fat pointer (ptr + vtable)
```

Fat pointers enable:
- Slices without heap allocation
- Trait objects without double indirection
- DSTs (Dynamically Sized Types) on the stack

The metadata is computed at coercion time, carried efficiently in registers.

## Connections
← [[011_trait_objects]]
← [[034_vtable]]
→ [[089_dst]]
→ [[090_slice_internals]]

---
Level: L4
Date: 2025-08-15
Tags: #pointers #fat-pointers #memory #slices