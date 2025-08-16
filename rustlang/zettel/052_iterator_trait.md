# Iterator Trait

## Core Insight
The Iterator trait defines a sequence abstraction with a single required method `next()`, while providing 70+ default methods for free.

```rust
trait Iterator {
    type Item;
    
    fn next(&mut self) -> Option<Self::Item>;
    
    // 70+ provided methods
    fn map<B, F>(self, f: F) -> Map<Self, F> {...}
    fn filter<P>(self, predicate: P) -> Filter<Self, P> {...}
    fn fold<B, F>(self, init: B, f: F) -> B {...}
    // ...
}
```

Custom iterator:
```rust
struct Counter {
    count: u32,
}

impl Iterator for Counter {
    type Item = u32;
    
    fn next(&mut self) -> Option<Self::Item> {
        self.count += 1;
        if self.count < 6 {
            Some(self.count)
        } else {
            None
        }
    }
}
```

Iterator consumption:
- `for` loops call `into_iter()`
- Adapters are lazy (map, filter)
- Consumers drive execution (collect, sum)

The trait's simplicity enables implementing iterators for any sequence-like type.

## Connections
← [[018_iterators]]
→ [[053_lazy_evaluation]]
→ [[110_into_iterator]]

---
Level: L4
Date: 2025-08-15
Tags: #iterator #trait #lazy #sequences