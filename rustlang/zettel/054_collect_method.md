# Collect Method

## Core Insight
`collect()` transforms iterators into collections, using type inference to determine the target collection type through the FromIterator trait.

```rust
// Type annotation determines collection
let vec: Vec<i32> = (1..5).collect();
let set: HashSet<i32> = (1..5).collect();
let string: String = ['h', 'e', 'l', 'l', 'o'].iter().collect();

// Turbofish syntax
let vec = (1..5).collect::<Vec<_>>();

// Collecting into Result/Option
let results: Result<Vec<_>, _> = 
    ["1", "2", "3"]
    .iter()
    .map(|s| s.parse::<i32>())
    .collect(); // Ok([1, 2, 3]) or Err on first failure
```

FromIterator trait:
```rust
trait FromIterator<A> {
    fn from_iter<T: IntoIterator<Item = A>>(iter: T) -> Self;
}
```

Collecting with capacity hint:
```rust
// Iterator size_hint helps pre-allocate
let vec: Vec<_> = (0..1000).collect(); // Pre-allocates 1000 elements
```

`collect()` is the bridge between lazy iterators and eager collections, forcing evaluation.

## Connections
← [[018_iterators]]
→ [[112_from_iterator]]
→ [[113_turbofish]]

---
Level: L3
Date: 2025-08-15
Tags: #iterators #collect #collections #type-inference