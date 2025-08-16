# Trait Bounds

## Core Insight
Trait bounds constrain generic types to those implementing specific traits, enabling generic code to use trait methods safely.

```rust
// Single bound
fn print_debug<T: Debug>(value: &T) {
    println!("{:?}", value);
}

// Multiple bounds with +
fn compare_and_display<T: Display + PartialOrd>(a: &T, b: &T) {
    if a > b {
        println!("{} is greater", a);
    }
}

// Where clause for complex bounds
fn complex<T, U>(t: &T, u: &U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{
    // Implementation
}
```

Bounds can be:
- Trait bounds: `T: Display`
- Lifetime bounds: `T: 'a`
- Higher-ranked: `for<'a> T: Trait<'a>`

Trait bounds enable zero-cost abstractions by constraining at compile time, generating specialized code via monomorphization.

## Connections
← [[009_trait_system]]
← [[010_generic_types]]
→ [[038_where_clauses]]
→ [[039_hrtb]]

---
Level: L4
Date: 2025-08-15
Tags: #trait #generics #bounds #constraints