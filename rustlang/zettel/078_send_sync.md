# Send and Sync

## Core Insight
Send and Sync marker traits encode thread safety at the type level - Send for moving between threads, Sync for sharing between threads.

```rust
// Send: Type can be transferred between threads
unsafe impl Send for MyType {}

// Sync: Type can be shared between threads (&T is Send)
unsafe impl Sync for MyType {}

// Relationships
impl<T: Sync> Send for &T {} // &T is Send if T is Sync
impl<T: Send> Sync for T {} // T is Sync if &mut T is Send
```

Auto implementation rules:
```rust
// Automatically Send if all fields are Send
struct Container {
    data: Vec<u8>,  // Send
    count: usize,   // Send
} // Container is Send

// Not Send due to raw pointer
struct NotSend {
    ptr: *const u8, // !Send
} // NotSend is !Send

// Rc is !Send (non-atomic refcount)
// Arc is Send (atomic refcount)
```

Thread safety requirements:
```rust
thread::spawn(|| {
    // Closure must be Send + 'static
});

fn share<T: Sync>(data: &T) {
    // Can be accessed from multiple threads
}
```

Send and Sync make thread safety a compile-time guarantee.

## Connections
← [[030_marker_traits]]
→ [[150_thread_safety]]
→ [[151_fearless_concurrency]]

---
Level: L4
Date: 2025-08-15
Tags: #send #sync #thread-safety #markers