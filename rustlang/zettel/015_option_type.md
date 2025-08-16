# Option Type

## Core Insight
Option<T> explicitly represents the presence or absence of a value, eliminating null pointer exceptions at compile time.

```rust
enum Option<T> {
    Some(T),
    None,
}

fn find_user(id: u32) -> Option<User> {
    if id == 1 {
        Some(User { id, name: "Alice".into() })
    } else {
        None
    }
}

// Must handle both cases
match find_user(1) {
    Some(user) => println!("Found: {}", user.name),
    None => println!("User not found"),
}
```

Combinators for functional composition:
```rust
let value = Some(5)
    .map(|x| x * 2)      // Some(10)
    .filter(|x| x > &5)  // Some(10)
    .and_then(|x| Some(x + 1)); // Some(11)
```

Option makes absence a first-class concept, forcing explicit handling and preventing "billion dollar mistake" null references.

## Connections
← [[014_result_type]]
→ [[043_null_safety]]
→ [[044_combinator_pattern]]

---
Level: L3
Date: 2025-08-15
Tags: #option #null-safety #types #monadic