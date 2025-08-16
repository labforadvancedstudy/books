# If Let

## Core Insight
`if let` provides syntactic sugar for matching a single pattern, eliminating boilerplate when only one case matters.

```rust
// Without if let
match some_option {
    Some(value) => println!("Got {}", value),
    None => (), // Boilerplate
}

// With if let
if let Some(value) = some_option {
    println!("Got {}", value);
}

// With else branch
if let Some(x) = some_option {
    println!("Got {}", x);
} else {
    println!("Got nothing");
}
```

Chaining with else if let:
```rust
if let Some(color) = favorite_color {
    println!("Using favorite: {:?}", color);
} else if let Ok(age) = age_string.parse::<u8>() {
    println!("Age-based color for {}", age);
} else {
    println!("Using default");
}
```

While let for loops:
```rust
while let Some(value) = iterator.next() {
    process(value);
}
```

`if let` trades exhaustiveness for conciseness - use when you only care about one pattern.

## Connections
← [[045_match_expression]]
← [[016_pattern_matching]]
→ [[103_let_else]]
→ [[104_while_let]]

---
Level: L3
Date: 2025-08-15
Tags: #if-let #pattern-matching #syntax-sugar #control-flow