# Higher-Ranked Trait Bounds (HRTB)

## Core Insight
HRTB with `for<'a>` syntax express that bounds must hold for all possible lifetimes, not just a specific one chosen at compile time.

```rust
// This function accepts closures that work with ANY lifetime
fn apply_to_ref<F>(f: F) 
where
    F: for<'a> Fn(&'a str) -> &'a str,
{
    let owned = String::from("hello");
    let result = f(&owned); // 'a is chosen here
    
    let static_str = "world";
    let result2 = f(static_str); // Different 'a chosen here
}

// Without HRTB - wouldn't work
fn bad_apply<'a, F>(f: F) // 'a fixed too early
where 
    F: Fn(&'a str) -> &'a str,
{
    let owned = String::from("hello");
    // let result = f(&owned); // ERROR: owned doesn't live for 'a
}
```

Common patterns:
```rust
// Implicitly uses HRTB
fn takes_fn(f: impl Fn(&str)) // Desugars to for<'a> Fn(&'a str)

// Explicit HRTB for complex cases
where T: for<'a> Deserialize<'a>
```

HRTB enables late-bound lifetimes, essential for higher-order functions.

## Connections
← [[027_higher_ranked_lifetimes]]
← [[038_where_clauses]]
→ [[075_late_bound_lifetimes]]

---
Level: L6
Date: 2025-08-15
Tags: #hrtb #lifetimes #bounds #higher-order