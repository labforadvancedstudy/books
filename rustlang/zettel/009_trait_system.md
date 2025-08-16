# Trait System

## Core Insight
Traits define shared behavior through interfaces, enabling polymorphism without inheritance hierarchies.

```rust
trait Draw {
    fn draw(&self);
}

struct Circle { radius: f64 }
struct Square { side: f64 }

impl Draw for Circle {
    fn draw(&self) {
        println!("Drawing circle with radius {}", self.radius);
    }
}

impl Draw for Square {
    fn draw(&self) {
        println!("Drawing square with side {}", self.side);
    }
}
```

Traits enable:
- Interface abstraction without inheritance
- Static dispatch (zero-cost) via generics
- Dynamic dispatch via trait objects (`dyn Trait`)
- Extension of external types (orphan rule permitting)

Trait bounds constrain generic types:
```rust
fn print_drawable<T: Draw>(item: &T) {
    item.draw(); // Static dispatch
}
```

## Connections
→ [[010_generic_types]]
→ [[011_trait_objects]]
→ [[012_associated_types]]
→ [[013_trait_bounds]]

---
Level: L4
Date: 2025-08-15
Tags: #trait #polymorphism #abstraction #interface