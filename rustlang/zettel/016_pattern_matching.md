# Pattern Matching

## Core Insight
Pattern matching destructures data and control flow simultaneously, making complex branching logic both exhaustive and expressive.

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(u8, u8, u8),
}

match message {
    Message::Quit => quit(),
    Message::Move { x, y } => move_cursor(x, y),
    Message::Write(text) => println!("{}", text),
    Message::ChangeColor(r, g, b) => set_color(r, g, b),
}
```

Patterns can be:
- Literals: `42`, `"hello"`
- Variables: `x` (binds value)
- Wildcards: `_` (ignores value)
- Destructuring: tuples, structs, enums
- Guards: `x if x > 5`
- Ranges: `1..=5`
- Or-patterns: `'a' | 'e' | 'i'`

```rust
let point = (3, 5);
match point {
    (0, y) => println!("On y axis at {}", y),
    (x, 0) => println!("On x axis at {}", x),
    _ => println!("Off axes"),
}
```

Exhaustiveness checking ensures all cases handled at compile time.

## Connections
→ [[045_match_expression]]
→ [[046_if_let]]
→ [[047_destructuring]]
→ [[048_pattern_guards]]

---
Level: L3
Date: 2025-08-15
Tags: #pattern-matching #control-flow #destructuring #exhaustiveness