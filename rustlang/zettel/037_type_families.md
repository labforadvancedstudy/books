# Type Families

## Core Insight
Associated types create type families where implementing a trait determines a related set of types, establishing type-level relationships.

```rust
trait Graph {
    type Node;
    type Edge;
    
    fn nodes(&self) -> Vec<Self::Node>;
    fn edges(&self) -> Vec<Self::Edge>;
}

struct GridGraph;

impl Graph for GridGraph {
    type Node = (i32, i32);  // Position
    type Edge = ((i32, i32), (i32, i32)); // Connection
    
    fn nodes(&self) -> Vec<Self::Node> {
        // Return grid positions
    }
}
```

Type families express relationships:
```rust
trait Collection {
    type Item;
    type Iter: Iterator<Item = Self::Item>;
    
    fn iter(&self) -> Self::Iter;
}
```

Benefits over generic parameters:
- One implementation per type (not per parameter combination)
- Cleaner APIs without parameter explosion
- Express complex type relationships
- Enable type projection: `<T as Graph>::Node`

Type families model domain relationships at the type level.

## Connections
← [[012_associated_types]]
→ [[033_gat]]
→ [[091_type_projection]]

---
Level: L5
Date: 2025-08-15
Tags: #types #associated-types #type-families #design