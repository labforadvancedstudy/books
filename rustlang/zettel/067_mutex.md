# Mutex

## Core Insight
Mutex provides thread-safe interior mutability through mutual exclusion, protecting shared data with locks that enforce exclusive access.

```rust
use std::sync::Mutex;

let counter = Mutex::new(0);

// Lock must be acquired to access data
{
    let mut guard = counter.lock().unwrap();
    *guard += 1;
} // Lock automatically released when guard drops

// Multiple threads
let counter = Arc::new(Mutex::new(0));
let mut handles = vec![];

for _ in 0..10 {
    let counter = Arc::clone(&counter);
    handles.push(thread::spawn(move || {
        let mut num = counter.lock().unwrap();
        *num += 1;
    }));
}
```

Mutex properties:
- Blocking: threads wait for lock
- Poisoning: panics poison the mutex
- RAII: lock released on drop
- Not reentrant: same thread can't lock twice

Deadlock prevention:
```rust
// Always acquire locks in same order
let (guard1, guard2) = if id1 < id2 {
    (mutex1.lock(), mutex2.lock())
} else {
    (mutex2.lock(), mutex1.lock())
};
```

Mutex trades performance for thread safety, essential for shared mutable state.

## Connections
← [[023_interior_mutability]]
→ [[068_arc_mutex_pattern]]
→ [[136_rwlock]]
→ [[137_parking_lot]]

---
Level: L4
Date: 2025-08-15
Tags: #mutex #concurrency #thread-safety #locks