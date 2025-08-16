# Lifetime Elision

## Core Insight
The compiler automatically infers lifetimes in common patterns, reducing annotation burden while maintaining safety.

Elision rules (applied in order):
1. Each input reference gets its own lifetime
2. If one input lifetime, output gets same lifetime
3. If `&self` or `&mut self`, output gets self's lifetime

```rust
// Written
fn first(s: &str) -> &str { &s[0..1] }

// Expanded by compiler
fn first<'a>(s: &'a str) -> &'a str { &s[0..1] }

// Multiple inputs - won't compile without explicit lifetimes
fn longest(x: &str, y: &str) -> &str { // ERROR
    // Compiler can't know which lifetime to use
}

// Must write explicitly
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}
```

Methods with elision:
```rust
impl Buffer {
    fn data(&self) -> &[u8] { // Elided: both have self's lifetime
        &self.internal_data
    }
}
```

Elision covers ~87% of cases in practice, explicit lifetimes only needed for complex relationships.

## Connections
← [[005_lifetime]]
→ [[027_higher_ranked_lifetimes]]
→ [[074_lifetime_bounds]]

---
Level: L4
Date: 2025-08-15
Tags: #lifetime #elision #inference #ergonomics