# Interior Mutability

## Core Insight
Interior mutability allows mutation through immutable references by moving borrow checking from compile-time to runtime.

```rust
use std::cell::RefCell;

struct Counter {
    value: RefCell<i32>,
}

impl Counter {
    fn increment(&self) { // Note: &self, not &mut self
        *self.value.borrow_mut() += 1;
    }
}
```

Types providing interior mutability:
- `Cell<T>`: Copy types, no runtime checks
- `RefCell<T>`: runtime borrow checking
- `Mutex<T>`: thread-safe with locking
- `RwLock<T>`: multiple readers XOR one writer
- `Atomic*`: lock-free concurrency

```rust
use std::cell::Cell;

let cell = Cell::new(5);
cell.set(10); // Mutate through &Cell
```

Runtime panics if borrow rules violated:
```rust
let ref_cell = RefCell::new(5);
let r1 = ref_cell.borrow();
let r2 = ref_cell.borrow_mut(); // PANIC: already borrowed
```

Enables patterns like shared ownership with mutation (Rc<RefCell<T>>).

## Connections
← [[002_borrowing]]
→ [[066_cell_types]]
→ [[067_mutex]]
→ [[068_arc_mutex_pattern]]

---
Level: L4
Date: 2025-08-15
Tags: #mutability #interior-mutability #runtime-checks #cell