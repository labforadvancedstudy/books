# Trait Objects

## Core Insight
Trait objects (`dyn Trait`) enable runtime polymorphism through dynamic dispatch, trading compile-time performance for runtime flexibility.

```rust
trait Draw {
    fn draw(&self);
}

// Trait object: type erased at compile time
let shapes: Vec<Box<dyn Draw>> = vec![
    Box::new(Circle { radius: 1.0 }),
    Box::new(Square { side: 2.0 }),
];

for shape in shapes {
    shape.draw(); // Dynamic dispatch via vtable
}
```

Trait objects use fat pointers:
- Data pointer → actual object
- VTable pointer → method implementations

Object-safe traits requirements:
- No generic methods
- No `Self` in return position
- No associated constants
- All methods dispatchable

Dynamic dispatch has runtime cost (vtable lookup) but enables heterogeneous collections and plugin architectures.

## Connections
← [[009_trait_system]]
→ [[034_vtable]]
→ [[035_object_safety]]
→ [[036_fat_pointers]]

---
Level: L4
Date: 2025-08-15
Tags: #trait #dynamic-dispatch #polymorphism #vtable