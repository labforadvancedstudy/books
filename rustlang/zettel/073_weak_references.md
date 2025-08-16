# Weak References

## Core Insight
Weak references break reference cycles by not contributing to the reference count, preventing memory leaks in graph-like structures.

```rust
use std::rc::{Rc, Weak};
use std::cell::RefCell;

struct Node {
    value: i32,
    parent: RefCell<Weak<Node>>, // Weak to avoid cycle
    children: RefCell<Vec<Rc<Node>>>,
}

let parent = Rc::new(Node {
    value: 5,
    parent: RefCell::new(Weak::new()),
    children: RefCell::new(vec![]),
});

let child = Rc::new(Node {
    value: 3,
    parent: RefCell::new(Rc::downgrade(&parent)), // Create weak
    children: RefCell::new(vec![]),
});

parent.children.borrow_mut().push(Rc::clone(&child));

// Accessing through weak reference
if let Some(parent) = child.parent.borrow().upgrade() {
    println!("Parent value: {}", parent.value);
}
```

Weak properties:
- Doesn't prevent dropping
- Must upgrade to access (returns Option<Rc<T>>)
- Can detect if value still exists
- Zero weak_count doesn't drop allocation

Common patterns:
- Parent-child relationships
- Observers/subscribers
- Cache entries
- Back-references in trees

## Connections
← [[072_rc_arc]]
→ [[143_rc_cycles]]
→ [[144_observer_pattern]]

---
Level: L4
Date: 2025-08-15
Tags: #weak-references #memory #cycles #smart-pointers