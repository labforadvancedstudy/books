# Question Mark Operator

## Core Insight
The `?` operator provides syntactic sugar for error propagation, automatically converting errors and returning early on failure.

```rust
// Without ?
fn read_number(path: &str) -> Result<i32, Box<dyn Error>> {
    match fs::read_to_string(path) {
        Ok(contents) => {
            match contents.trim().parse::<i32>() {
                Ok(num) => Ok(num),
                Err(e) => Err(Box::new(e)),
            }
        }
        Err(e) => Err(Box::new(e)),
    }
}

// With ?
fn read_number(path: &str) -> Result<i32, Box<dyn Error>> {
    let contents = fs::read_to_string(path)?;
    let num = contents.trim().parse::<i32>()?;
    Ok(num)
}
```

How `?` works:
1. If `Ok(value)`, unwraps and continues
2. If `Err(e)`, calls `From::from(e)` and returns early

Works with Option too:
```rust
fn get_length(s: Option<String>) -> Option<usize> {
    Some(s?.len()) // Returns None if s is None
}
```

The `?` operator uses the `Try` trait, making it extensible to custom types.

## Connections
← [[014_result_type]]
← [[040_error_handling]]
→ [[042_try_trait]]
→ [[095_from_trait]]

---
Level: L3
Date: 2025-08-15
Tags: #error-handling #operators #syntactic-sugar #propagation