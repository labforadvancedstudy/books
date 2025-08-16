# Result Type

## Core Insight
Result<T, E> makes error handling explicit in the type system, forcing developers to handle failures and eliminating hidden error states.

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}

fn divide(a: f64, b: f64) -> Result<f64, String> {
    if b == 0.0 {
        Err(String::from("Division by zero"))
    } else {
        Ok(a / b)
    }
}

// Explicit error handling required
match divide(10.0, 2.0) {
    Ok(result) => println!("Result: {}", result),
    Err(e) => println!("Error: {}", e),
}
```

The `?` operator provides early return propagation:
```rust
fn calculate() -> Result<f64, String> {
    let x = divide(10.0, 2.0)?; // Returns early if Err
    let y = divide(x, 5.0)?;
    Ok(y * 2.0)
}
```

Result eliminates exceptions, making error paths visible in function signatures and ensuring errors cannot be ignored silently.

## Connections
→ [[015_option_type]]
→ [[040_error_handling]]
→ [[041_question_mark]]
→ [[042_try_trait]]

---
Level: L3
Date: 2025-08-15
Tags: #error-handling #result #types #monadic