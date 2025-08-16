# Move Keyword

## Core Insight
The `move` keyword forces a closure to take ownership of captured values, even when borrowing would suffice, enabling closures to outlive their environment.

```rust
let data = vec![1, 2, 3];

// Without move - borrows data
let closure = || println!("{:?}", data);

// With move - takes ownership
let closure = move || println!("{:?}", data);
// data is no longer accessible here

// Essential for returning closures
fn make_adder(x: i32) -> impl Fn(i32) -> i32 {
    move |y| x + y // Must move x, as it won't exist after function returns
}
```

Threading requires move:
```rust
let data = vec![1, 2, 3];
thread::spawn(move || {
    // Thread might outlive data's scope, so must own it
    println!("{:?}", data);
});
```

Move with Copy types:
```rust
let x = 5; // i32 implements Copy
let closure = move || x * 2; // Copies x, not moves
println!("{}", x); // x still valid!
```

`move` decouples closure lifetime from captured variable lifetime.

## Connections
← [[017_closures]]
← [[049_fn_traits]]
→ [[108_closure_capture]]

---
Level: L4
Date: 2025-08-15
Tags: #move #closures #ownership #lifetime