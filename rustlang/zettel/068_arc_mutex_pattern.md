# Arc<Mutex<T>> Pattern

## Core Insight
Arc<Mutex<T>> combines atomic reference counting with mutual exclusion, enabling safe shared mutable state across threads.

```rust
use std::sync::{Arc, Mutex};
use std::thread;

let data = Arc::new(Mutex::new(vec![1, 2, 3]));
let mut handles = vec![];

for i in 0..3 {
    let data_clone = Arc::clone(&data);
    let handle = thread::spawn(move || {
        let mut vec = data_clone.lock().unwrap();
        vec.push(i);
    });
    handles.push(handle);
}

for handle in handles {
    handle.join().unwrap();
}

println!("{:?}", *data.lock().unwrap()); // [1, 2, 3, 0, 1, 2]
```

Why both Arc and Mutex:
- `Arc`: shared ownership across threads
- `Mutex`: exclusive access to data
- Together: multiple owners, one accessor

Common patterns:
```rust
// Shared configuration
type SharedConfig = Arc<Mutex<Config>>;

// Thread-safe counter
type Counter = Arc<Mutex<usize>>;

// Shared cache
type Cache<K, V> = Arc<Mutex<HashMap<K, V>>>;
```

Performance consideration: Consider `RwLock` for read-heavy workloads, or lock-free structures for high contention.

## Connections
← [[067_mutex]]
← [[072_rc_arc]]
→ [[138_dashmap]]
→ [[139_lock_free]]

---
Level: L4
Date: 2025-08-15
Tags: #arc #mutex #concurrency #shared-state