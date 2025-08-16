# VTable

## Core Insight
Virtual tables store function pointers for dynamic dispatch, enabling runtime polymorphism with a small indirection cost.

VTable structure for trait object:
```rust
// For dyn Draw trait object
struct DrawVTable {
    drop: fn(*mut ()),           // Destructor
    size: usize,                 // Size of type
    align: usize,                // Alignment requirement
    draw: fn(*const ()),         // Trait method
}

// Fat pointer representation
struct DynDraw {
    data: *const (),  // Pointer to actual object
    vtable: *const DrawVTable, // Pointer to vtable
}
```

Dynamic dispatch process:
1. Load vtable pointer from trait object
2. Load function pointer from vtable
3. Call function with data pointer

```rust
trait Animal {
    fn speak(&self);
}

let animal: &dyn Animal = &Dog;
animal.speak(); 
// Compiles to: (*animal.vtable.speak)(animal.data)
```

Cost: One pointer indirection + no inlining. VTables are generated once per type per trait, shared across all instances.

## Connections
← [[011_trait_objects]]
→ [[035_object_safety]]
→ [[036_fat_pointers]]
→ [[087_dynamic_dispatch]]

---
Level: L5
Date: 2025-08-15
Tags: #vtable #dynamic-dispatch #trait-objects #runtime