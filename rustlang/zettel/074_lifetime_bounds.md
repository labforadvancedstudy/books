# Lifetime Bounds

## Core Insight
Lifetime bounds constrain type parameters and associated types to outlive specific lifetimes, ensuring references remain valid.

```rust
// T must live at least as long as 'a
struct Ref<'a, T: 'a> {
    data: &'a T,
}

// Function with lifetime bounds
fn process<'a, T>(data: &'a T) 
where
    T: 'a + Debug, // T must outlive 'a AND implement Debug
{
    println!("{:?}", data);
}

// Trait with lifetime bounds
trait Container<'a> {
    type Item: 'a; // Associated type must outlive 'a
}

// Static lifetime bound
fn require_static<T: 'static>(value: T) {
    // T contains no non-'static references
}
```

Common patterns:
```rust
// Ensure closure doesn't outlive captures
fn with_closure<'a, F>(data: &'a str, f: F)
where
    F: FnOnce() + 'a,
{
    f();
}

// Thread spawning requires 'static
thread::spawn(move || { // Closure must be 'static
    // No borrowed data unless 'static
});
```

Lifetime bounds prevent dangling references in generic code by encoding lifetime relationships in type signatures.

## Connections
← [[005_lifetime]]
← [[026_lifetime_elision]]
→ [[145_variance]]

---
Level: L5
Date: 2025-08-15
Tags: #lifetime #bounds #generics #constraints