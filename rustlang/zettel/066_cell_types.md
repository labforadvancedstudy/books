# Cell Types

## Core Insight
Cell and RefCell provide interior mutability for single-threaded contexts, moving borrow checking from compile-time to runtime or avoiding it entirely.

```rust
use std::cell::{Cell, RefCell};

// Cell: Copy types, no runtime checks
struct Counter {
    count: Cell<i32>,
}

impl Counter {
    fn increment(&self) { // Note: &self, not &mut self
        self.count.set(self.count.get() + 1);
    }
}

// RefCell: runtime borrow checking
struct Logger {
    messages: RefCell<Vec<String>>,
}

impl Logger {
    fn log(&self, msg: String) {
        self.messages.borrow_mut().push(msg);
    }
    
    fn get_messages(&self) -> Vec<String> {
        self.messages.borrow().clone()
    }
}
```

Cell vs RefCell:
- `Cell<T>`: T must be Copy, get/set only
- `RefCell<T>`: any T, runtime borrow checks
- `UnsafeCell<T>`: raw interior mutability

Runtime panics with RefCell:
```rust
let cell = RefCell::new(5);
let b1 = cell.borrow();
let b2 = cell.borrow_mut(); // PANIC: already borrowed
```

Enables mutation patterns impossible with static borrowing.

## Connections
← [[023_interior_mutability]]
→ [[134_unsafe_cell]]
→ [[135_once_cell]]

---
Level: L4
Date: 2025-08-15
Tags: #cell #interior-mutability #runtime-checks #single-threaded