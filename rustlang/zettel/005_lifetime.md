# Lifetime

## Core Insight
Lifetimes are compile-time annotations that describe how long references remain valid, preventing dangling references without runtime cost.

Every reference has a lifetime, usually inferred:

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}

// Lifetime 'a means: returned reference lives as long as
// the shorter of x and y's lifetimes
```

Lifetime annotations don't change how long values live - they describe relationships between lifetimes:

```rust
struct Book<'a> {
    title: &'a str, // Book cannot outlive the string it references
}
```

The borrow checker uses lifetimes to ensure:
- No dangling references
- References always valid
- Memory safety without GC

`'static` lifetime = entire program duration (string literals, leaked memory).

## Connections
← [[002_borrowing]]
← [[001_ownership]]
→ [[006_borrow_checker]]
→ [[026_lifetime_elision]]
→ [[027_higher_ranked_lifetimes]]

---
Level: L4
Date: 2025-08-15
Tags: #lifetime #references #safety #memory