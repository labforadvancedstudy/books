# Combinator Pattern

## Core Insight
Combinators compose operations on wrapped values without unwrapping, enabling functional pipelines that handle absence and errors elegantly.

Option combinators:
```rust
let value = Some(5)
    .map(|x| x * 2)          // Some(10)
    .filter(|&x| x > 5)      // Some(10)
    .and_then(|x| Some(x/2)) // Some(5)
    .or(Some(0))             // Some(5)
    .unwrap_or(0);           // 5
```

Result combinators:
```rust
let result = read_file("data.txt")
    .map(|content| content.trim().to_string())
    .and_then(|s| s.parse::<i32>())
    .map_err(|e| format!("Error: {}", e))
    .unwrap_or_else(|_| 0);
```

Common combinators:
- `map`: transform inner value
- `and_then`/`flat_map`: chain operations that return wrapped values
- `filter`: conditional keeping
- `or_else`: provide alternative on None/Err
- `zip`: combine multiple wrapped values

Combinators eliminate nested match statements, creating readable data transformation pipelines.

## Connections
← [[015_option_type]]
← [[014_result_type]]
→ [[100_monad_pattern]]
→ [[101_railway_programming]]

---
Level: L4
Date: 2025-08-15
Tags: #combinators #functional #patterns #composition