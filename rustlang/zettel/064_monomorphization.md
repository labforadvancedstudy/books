# Monomorphization

## Core Insight
Monomorphization generates specialized code for each concrete type used with generics, trading compile time and binary size for runtime performance.

```rust
fn generic<T: Display>(val: T) {
    println!("{}", val);
}

generic(5);          // Generates generic::<i32>
generic("hello");    // Generates generic::<&str>
generic(3.14);       // Generates generic::<f64>

// Compiler generates three separate functions
```

Binary bloat vs performance:
```rust
// This creates one function per T type used
fn process<T: Clone>(items: Vec<T>) { /* ... */ }

// This creates only one function (dynamic dispatch)
fn process(items: Vec<Box<dyn Clone>>) { /* ... */ }
```

Monomorphization benefits:
- No vtable lookups
- Enables inlining
- Allows optimizations per type
- Zero abstraction cost

Downsides:
- Increased compile time
- Larger binary size
- No dynamic linking of generics

LLVM deduplicates identical monomorphizations, but each unique instantiation adds code.

## Connections
← [[010_generic_types]]
← [[022_zero_cost_abstractions]]
→ [[129_code_bloat]]
→ [[130_thin_lto]]

---
Level: L5
Date: 2025-08-15
Tags: #monomorphization #generics #compilation #optimization