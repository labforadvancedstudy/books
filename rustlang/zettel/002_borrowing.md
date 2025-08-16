# Borrowing

## Core Insight
Borrowing allows multiple parts of code to access data without taking ownership, enforced by the borrow checker at compile time.

Rust allows two types of borrows:
- Immutable references (`&T`): multiple allowed simultaneously
- Mutable references (`&mut T`): exactly one allowed, no other references

```rust
let mut s = String::from("hello");
let r1 = &s; // immutable borrow
let r2 = &s; // another immutable borrow - OK
// let r3 = &mut s; // ERROR: cannot borrow as mutable

let r3 = &mut s; // mutable borrow - OK after r1, r2 scope ends
```

The borrow checker ensures:
- No data races (mutable XOR immutable)
- No dangling references
- References always valid

This is Rust's answer to the shared mutability problem that plagues concurrent programming.

## Connections
← [[001_ownership]]
→ [[005_lifetime]]
→ [[006_borrow_checker]]
→ [[023_interior_mutability]]

---
Level: L3
Date: 2025-08-15
Tags: #memory #safety #references #borrowing