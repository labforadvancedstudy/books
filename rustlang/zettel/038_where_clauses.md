# Where Clauses

## Core Insight
Where clauses provide cleaner syntax for complex trait bounds, improving readability and enabling bounds that can't be expressed inline.

```rust
// Without where clause - hard to read
fn complex<T: Display + Clone, U: Debug + Clone>(t: T, u: U) {}

// With where clause - much cleaner
fn complex<T, U>(t: T, u: U)
where
    T: Display + Clone,
    U: Debug + Clone,
{}
```

Where clauses enable complex bounds:
```rust
// Bounds on associated types
fn process<I>(iter: I)
where
    I: Iterator,
    I::Item: Display,
{}

// Multiple type relationships
fn merge<T>(first: T, second: T) -> T
where
    T: Add<Output = T> + Copy,
{}

// Lifetime bounds
fn outlives<'a, T>(x: &'a T)
where
    T: 'a + Debug,
{}
```

Where clauses can appear on:
- Functions
- Impl blocks
- Traits
- Type aliases (with GATs)

They're not just syntax sugar - some bounds can only be expressed with where clauses.

## Connections
← [[013_trait_bounds]]
→ [[039_hrtb]]
→ [[092_implied_bounds]]

---
Level: L4
Date: 2025-08-15
Tags: #where-clause #bounds #syntax #generics