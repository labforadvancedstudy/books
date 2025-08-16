# Error Handling

## Core Insight
Rust's error handling philosophy makes failures explicit in types, recoverable via Result, and unrecoverable via panic, with zero runtime cost.

Error handling strategies:

```rust
// Recoverable errors with Result
fn read_file(path: &str) -> Result<String, io::Error> {
    fs::read_to_string(path)
}

// Unrecoverable errors with panic
fn get_element(vec: &Vec<i32>, index: usize) -> i32 {
    vec[index] // Panics on out-of-bounds
}

// Custom error types
#[derive(Debug)]
enum AppError {
    Io(io::Error),
    Parse(ParseIntError),
    Custom(String),
}

// Error propagation with ?
fn process() -> Result<i32, AppError> {
    let content = read_file("data.txt")?;
    let number = content.parse::<i32>()?;
    Ok(number * 2)
}
```

Best practices:
- Use `Result` for expected failures
- Use `panic!` for bugs/invariant violations
- Create domain-specific error types
- Implement `Error` trait for custom errors
- Use `anyhow` or `thiserror` for ergonomics

No hidden control flow - errors always visible in signatures.

## Connections
← [[014_result_type]]
→ [[041_question_mark]]
→ [[093_error_trait]]
→ [[094_panic_handling]]

---
Level: L4
Date: 2025-08-15
Tags: #error-handling #result #panic #exceptions