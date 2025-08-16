# Late Bound Lifetimes

## Core Insight
Late-bound lifetimes are decided at the call site rather than when the function is defined, enabling more flexible lifetime polymorphism.

```rust
// Late-bound: 'a decided when function is called
fn late_bound<'a>(x: &'a str) -> &'a str {
    x
}

// Early-bound: 'a part of function's type
fn early_bound<'a: 'static>(x: &'a str) -> &'a str {
    x
}

// Difference in usage
let f1 = late_bound; // Works: 'a not yet decided
// let f2 = early_bound; // Error: 'a must be specified
```

When lifetimes are late-bound:
- Appear only in function parameters
- No lifetime bounds
- Not used in where clauses

Impact on higher-order functions:
```rust
// Works because 'a is late-bound
fn apply<F>(f: F, s: &str) -> &str
where
    F: for<'a> Fn(&'a str) -> &'a str,
{
    f(s) // 'a chosen here
}

// Without late-binding, wouldn't work
fn apply_bad<'a, F>(f: F, s: &'a str) -> &'a str
where
    F: Fn(&'a str) -> &'a str, // 'a fixed too early
{
    f(s)
}
```

Late-bound lifetimes enable functions to be more polymorphic over lifetimes.

## Connections
← [[027_higher_ranked_lifetimes]]
← [[039_hrtb]]
→ [[146_early_bound]]

---
Level: L6
Date: 2025-08-15
Tags: #lifetimes #late-bound #polymorphism #functions