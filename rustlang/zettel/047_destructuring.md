# Destructuring

## Core Insight
Destructuring breaks apart compound data types into their components, binding values to variables in a single pattern.

```rust
// Tuple destructuring
let (x, y, z) = (1, 2, 3);

// Struct destructuring
struct Point { x: i32, y: i32 }
let Point { x, y } = point;

// With renaming
let Point { x: horizontal, y: vertical } = point;

// Nested destructuring
let ((a, b), Point { x, y }) = ((1, 2), Point { x: 3, y: 4 });

// Enum destructuring
enum Message {
    Move { x: i32, y: i32 },
    Write(String),
}

match msg {
    Message::Move { x, y } => println!("Move to {}, {}", x, y),
    Message::Write(text) => println!("Text: {}", text),
}
```

Partial destructuring with `..`:
```rust
let Point { x, .. } = point; // Ignore other fields
let (first, .., last) = (1, 2, 3, 4, 5); // first=1, last=5
```

Destructuring in function parameters:
```rust
fn print_point(&Point { x, y }: &Point) {
    println!("({}, {})", x, y);
}
```

## Connections
← [[016_pattern_matching]]
→ [[105_structural_patterns]]
→ [[106_rest_patterns]]

---
Level: L3
Date: 2025-08-15
Tags: #destructuring #pattern-matching #syntax #bindings