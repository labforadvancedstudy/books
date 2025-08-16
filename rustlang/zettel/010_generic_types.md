# Generic Types

## Core Insight
Generics enable type parametrization with zero runtime cost through monomorphization - the compiler generates specialized code for each concrete type.

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn new(x: T, y: T) -> Self {
        Point { x, y }
    }
}

// Compiler generates separate implementations:
let int_point = Point::new(5, 10);      // Point<i32>
let float_point = Point::new(1.0, 4.0); // Point<f64>
```

Generic functions with trait bounds:
```rust
fn largest<T: PartialOrd>(list: &[T]) -> &T {
    let mut largest = &list[0];
    for item in list {
        if item > largest {
            largest = item;
        }
    }
    largest
}
```

Monomorphization means generic code has no runtime overhead - it's as fast as hand-written specialized versions.

## Connections
← [[009_trait_system]]
→ [[013_trait_bounds]]
→ [[032_const_generics]]
→ [[033_gat]]

---
Level: L4
Date: 2025-08-15
Tags: #generics #types #monomorphization #zero-cost