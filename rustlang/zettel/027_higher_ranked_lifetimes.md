# Higher Ranked Lifetimes

## Core Insight
`for<'a>` syntax allows expressing that a trait bound must hold for all possible lifetimes, enabling more flexible generic code.

```rust
// Function that works with any lifetime
fn example<F>(f: F)
where
    F: for<'a> Fn(&'a str) -> &'a str,
{
    let s = String::from("hello");
    let result = f(&s); // Works with any lifetime
}

// Without HRTB, this wouldn't work:
fn bad_example<'a, F>(f: F)  // 'a fixed at call site
where
    F: Fn(&'a str) -> &'a str,
{
    let s = String::from("hello");
    // let result = f(&s); // ERROR: 's' doesn't live for 'a
}
```

Common with closure bounds:
```rust
// Closure must work for ANY lifetime, not a specific one
fn filter_map<F>(f: F) 
where
    F: for<'a> FnMut(&'a Item) -> Option<&'a str>
```

HRTB is implicitly added for common patterns:
```rust
fn takes_closure(f: impl Fn(&str)) // Desugars to:
fn takes_closure<F>(f: F) where F: for<'a> Fn(&'a str)
```

Enables late-bound lifetimes, decided at use site rather than declaration.

## Connections
← [[005_lifetime]]
← [[026_lifetime_elision]]
→ [[039_hrtb]]
→ [[075_late_bound_lifetimes]]

---
Level: L6
Date: 2025-08-15
Tags: #lifetime #hrtb #higher-ranked #generics