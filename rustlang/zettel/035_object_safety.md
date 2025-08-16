# Object Safety

## Core Insight
Object-safe traits can be used as trait objects (`dyn Trait`) by ensuring all methods can be called through vtable dispatch.

Object-safe requirements:
```rust
// Object-safe trait
trait Draw {
    fn draw(&self); // OK: self by reference
    fn color(&self) -> Color; // OK: no generics
}

// NOT object-safe
trait NotObjectSafe {
    fn generic<T>(&self, x: T); // Generic method
    fn returns_self(self) -> Self; // Returns Self
    fn no_self(); // No self parameter
    const CONST: i32; // Associated const
}
```

Rules for object safety:
1. No generic methods (can't monomorphize at runtime)
2. No `Self` in return position (size unknown)
3. All methods must have `self` receiver
4. No associated constants
5. No `where Self: Sized` bounds

Making traits object-safe:
```rust
trait MyTrait {
    fn safe_method(&self);
    
    fn not_safe() where Self: Sized {
        // Excluded from trait object
    }
}
```

Object safety ensures vtable can be constructed at compile time.

## Connections
← [[011_trait_objects]]
← [[034_vtable]]
→ [[088_self_types]]

---
Level: L5
Date: 2025-08-15
Tags: #object-safety #trait-objects #vtable #dynamic-dispatch