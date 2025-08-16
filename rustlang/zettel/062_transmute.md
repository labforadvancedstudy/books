# Transmute

## Core Insight
`transmute` reinterprets bits from one type to another with zero runtime cost, the most dangerous tool in unsafe Rust.

```rust
use std::mem::transmute;

unsafe {
    // Reinterpret u32 as i32
    let x: u32 = 42;
    let y: i32 = transmute(x); // Same bits, different type
    
    // Convert reference to raw pointer
    let r: &i32 = &42;
    let p: *const i32 = transmute(r);
}
```

Dangerous transmutes:
```rust
// UNDEFINED BEHAVIOR examples:
unsafe {
    let x: &i32 = transmute(0usize); // Null reference!
    let b: bool = transmute(2u8);    // Invalid bool!
    let f: fn() = transmute(0usize); // Null function pointer!
}
```

Safe alternatives:
```rust
// Prefer these over transmute:
let y = x as i32;                    // For numeric casts
let p = r as *const i32;             // For pointer casts
let bytes = u32::to_ne_bytes(x);     // For byte conversion
let f = f32::from_bits(bits);        // For float reinterpretation
```

Rules:
- Types must have same size
- Alignment must be compatible
- Result must be valid for target type

Transmute is last resort - almost always a safer alternative exists.

## Connections
← [[021_unsafe_rust]]
→ [[125_type_punning]]
→ [[126_bytemuck]]

---
Level: L6
Date: 2025-08-15
Tags: #transmute #unsafe #reinterpret #dangerous