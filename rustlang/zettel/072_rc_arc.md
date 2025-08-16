# Rc and Arc

## Core Insight
Rc (single-threaded) and Arc (thread-safe) provide shared ownership through reference counting, enabling multiple owners of the same data.

```rust
use std::rc::Rc;
use std::sync::Arc;

// Rc: single-threaded reference counting
let rc1 = Rc::new(vec![1, 2, 3]);
let rc2 = Rc::clone(&rc1); // Cheap: increments count
let rc3 = Rc::clone(&rc1);
assert_eq!(Rc::strong_count(&rc1), 3);
// Data dropped when count reaches 0

// Arc: atomic reference counting (thread-safe)
let arc = Arc::new(5);
let arc2 = Arc::clone(&arc);
thread::spawn(move || {
    println!("{}", arc2); // Can send across threads
});
```

Key differences:
```rust
// Rc: cheaper, single-threaded only
size_of::<Rc<T>>() == size_of::<*const T>()

// Arc: atomic operations, thread-safe
size_of::<Arc<T>>() == size_of::<*const T>()
// But uses atomic increment/decrement
```

Cycles cause leaks:
```rust
// Creates reference cycle - memory leak!
struct Node {
    next: Option<Rc<RefCell<Node>>>,
}
// Use Weak<T> to break cycles
```

Reference counting provides shared ownership with deterministic cleanup.

## Connections
← [[025_smart_pointers]]
→ [[073_weak_references]]
→ [[143_rc_cycles]]

---
Level: L4
Date: 2025-08-15
Tags: #rc #arc #reference-counting #shared-ownership