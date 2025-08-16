# Smart Pointers

## Core Insight
Smart pointers own their data and provide automatic memory management through RAII, extending regular pointers with metadata and behavior.

Common smart pointers:

```rust
// Box<T>: heap allocation
let boxed = Box::new(5);

// Rc<T>: reference counting (single-threaded)
let rc1 = Rc::new(vec![1, 2, 3]);
let rc2 = Rc::clone(&rc1); // Cheap: increments count

// Arc<T>: atomic reference counting (thread-safe)
let arc = Arc::new(Mutex::new(0));
let arc2 = Arc::clone(&arc);

// Cow<T>: Clone-on-write
let cow: Cow<str> = Cow::Borrowed("hello");
```

Smart pointers implement:
- `Deref` trait: automatic dereferencing
- `Drop` trait: cleanup logic
- Sometimes `DerefMut`: mutable dereferencing

```rust
struct MyBox<T>(T);

impl<T> Deref for MyBox<T> {
    type Target = T;
    fn deref(&self) -> &T {
        &self.0
    }
}
```

Enable patterns impossible with plain references: shared ownership, weak references, interior mutability.

## Connections
← [[004_drop_trait]]
← [[024_raii_pattern]]
→ [[071_box_type]]
→ [[072_rc_arc]]
→ [[073_weak_references]]

---
Level: L4
Date: 2025-08-15
Tags: #smart-pointers #memory #heap #reference-counting