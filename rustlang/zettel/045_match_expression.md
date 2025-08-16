# Match Expression

## Core Insight
Match is an expression that pattern matches exhaustively, returning a value and enforcing all cases are handled at compile time.

```rust
// Match is an expression - it returns a value
let description = match value {
    0 => "zero",
    1..=9 => "single digit",
    10..=99 => "double digit",
    _ => "large number",
};

// Exhaustiveness checking
enum Color { Red, Green, Blue }

let hex = match color {
    Color::Red => 0xFF0000,
    Color::Green => 0x00FF00,
    // Compile error if Blue not handled
    Color::Blue => 0x0000FF,
};
```

Advanced patterns:
```rust
match point {
    Point { x: 0, y } => println!("On y-axis at {}", y),
    Point { x, y: 0 } => println!("On x-axis at {}", x),
    Point { x, y } if x == y => println!("On diagonal"),
    Point { x, y } => println!("At ({}, {})", x, y),
}
```

Match ensures:
- All patterns same type
- Exhaustive coverage
- First matching arm wins
- Unreachable patterns warned

The compiler optimizes matches into jump tables or if-else chains.

## Connections
← [[016_pattern_matching]]
→ [[046_if_let]]
→ [[048_pattern_guards]]
→ [[102_match_ergonomics]]

---
Level: L3
Date: 2025-08-15
Tags: #match #pattern-matching #expressions #control-flow