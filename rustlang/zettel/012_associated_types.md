# Associated Types

## Core Insight
Associated types define type relationships within traits, providing cleaner APIs than generic parameters when there's only one logical implementation.

```rust
trait Iterator {
    type Item; // Associated type
    
    fn next(&mut self) -> Option<Self::Item>;
}

struct Counter {
    count: u32,
}

impl Iterator for Counter {
    type Item = u32; // Concrete type for this implementation
    
    fn next(&mut self) -> Option<Self::Item> {
        self.count += 1;
        Some(self.count)
    }
}
```

Associated types vs generic parameters:
```rust
// Generic parameter: multiple implementations possible
trait Container<T> {
    fn get(&self) -> T;
}

// Associated type: one implementation per type
trait Container {
    type Item;
    fn get(&self) -> Self::Item;
}
```

Associated types simplify trait usage when the relationship is one-to-one, avoiding type parameter proliferation.

## Connections
← [[009_trait_system]]
→ [[033_gat]]
→ [[037_type_families]]

---
Level: L5
Date: 2025-08-15
Tags: #trait #types #associated-types #api-design