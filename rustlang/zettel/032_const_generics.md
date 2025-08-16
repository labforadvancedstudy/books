# Const Generics

## Core Insight
Const generics allow types and functions to be parameterized by constant values, enabling compile-time sized arrays and zero-cost abstractions.

```rust
struct Array<T, const N: usize> {
    data: [T; N],
}

impl<T, const N: usize> Array<T, N> {
    fn new(value: T) -> Self 
    where T: Copy 
    {
        Array {
            data: [value; N], // N known at compile time
        }
    }
}

let arr: Array<i32, 5> = Array::new(0);
// Size is part of the type
```

Enables previously impossible patterns:
```rust
fn dot_product<const N: usize>(a: [f64; N], b: [f64; N]) -> f64 {
    a.iter().zip(b.iter()).map(|(x, y)| x * y).sum()
}

// Works with any size, verified at compile time
let result = dot_product([1.0, 2.0], [3.0, 4.0]);
```

Const generic parameters:
- Must be const-evaluable
- Can be integers, `bool`, `char`
- Cannot be strings or floats (yet)

Eliminates runtime size checks while maintaining type safety.

## Connections
← [[010_generic_types]]
→ [[033_gat]]
→ [[083_const_evaluation]]
→ [[084_min_const_generics]]

---
Level: L5
Date: 2025-08-15
Tags: #generics #const #compile-time #arrays