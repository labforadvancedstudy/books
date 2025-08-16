# Null Safety

## Core Insight
Rust eliminates null references entirely by encoding absence in the type system via Option, preventing the "billion dollar mistake."

No null in Rust:
```rust
// This doesn't exist in Rust:
// let x: String = null; // INVALID

// Instead, use Option:
let x: Option<String> = None;
let y: Option<String> = Some(String::from("hello"));

// Must handle both cases explicitly
match x {
    Some(s) => println!("Got: {}", s),
    None => println!("Got nothing"),
}
```

Null pointer optimization:
```rust
// Option<&T> has same size as &T
size_of::<Option<&i32>>() == size_of::<&i32>()
// Uses null pointer internally for None, but safely
```

Benefits:
- No null pointer exceptions
- Explicit absence handling
- Composable with combinators
- Zero runtime cost (optimized representation)

References in Rust are always valid - they cannot be null. This guarantee is enforced at compile time.

## Connections
← [[015_option_type]]
→ [[098_non_null]]
→ [[099_null_pointer_optimization]]

---
Level: L3
Date: 2025-08-15
Tags: #null-safety #option #memory-safety #type-system