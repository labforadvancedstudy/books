# Pattern Guards

## Core Insight
Pattern guards add boolean conditions to match arms, enabling complex matching logic that patterns alone cannot express.

```rust
let number = Some(4);

match number {
    Some(x) if x < 5 => println!("Less than five: {}", x),
    Some(x) if x == 5 => println!("Five exactly"),
    Some(x) => println!("Greater than five: {}", x),
    None => println!("No value"),
}
```

Guards can access bound variables:
```rust
enum Message {
    Hello { id: i32 },
}

match msg {
    Message::Hello { id } if id % 2 == 0 => {
        println!("Even ID: {}", id);
    }
    Message::Hello { id } => {
        println!("Odd ID: {}", id);
    }
}
```

Combining multiple patterns with guards:
```rust
match (x, y) {
    (1, y) | (x, 1) if x + y == 10 => println!("Sum is 10"),
    (x, y) if x == y => println!("Equal"),
    _ => println!("Other"),
}
```

Guards run after pattern matching succeeds, can call functions, and don't affect exhaustiveness checking.

## Connections
← [[045_match_expression]]
← [[016_pattern_matching]]
→ [[107_or_patterns]]

---
Level: L3
Date: 2025-08-15
Tags: #pattern-guards #matching #conditionals #control-flow