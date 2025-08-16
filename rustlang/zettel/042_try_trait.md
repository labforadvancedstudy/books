# Try Trait

## Core Insight
The Try trait abstracts over types that can be used with `?`, enabling custom types to support early return propagation.

```rust
// Simplified Try trait (actual is more complex)
trait Try {
    type Output;
    type Residual;
    
    fn from_output(output: Self::Output) -> Self;
    fn branch(self) -> ControlFlow<Self::Residual, Self::Output>;
}

enum ControlFlow<B, C> {
    Continue(C),
    Break(B),
}
```

Built-in implementations:
- `Result<T, E>`: continues with `T`, breaks with `E`
- `Option<T>`: continues with `T`, breaks with `None`
- `ControlFlow<B, C>`: explicit control flow

Custom implementation example:
```rust
enum Validation<T> {
    Valid(T),
    Invalid(String),
}

// Implement Try to use with ?
impl Try for Validation<T> {
    // Implementation details...
}

fn validate() -> Validation<i32> {
    let x = some_validation()?; // Now works!
    Valid(x * 2)
}
```

Try trait unifies early-return patterns across different types, making `?` extensible.

## Connections
← [[041_question_mark]]
→ [[096_control_flow]]
→ [[097_residual_types]]

---
Level: L5
Date: 2025-08-15
Tags: #try-trait #control-flow #operators #traits