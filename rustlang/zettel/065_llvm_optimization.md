# LLVM Optimization

## Core Insight
Rust leverages LLVM's optimization passes to transform high-level abstractions into efficient machine code, enabling zero-cost abstractions.

Optimization levels:
```rust
// cargo build              (opt-level = 0) - no optimization
// cargo build --release    (opt-level = 3) - full optimization
// opt-level = "z"          - optimize for size
// opt-level = "s"          - optimize for size, less aggressive
```

Key LLVM optimizations:
```rust
// Dead code elimination
if false {
    expensive_function(); // Removed entirely
}

// Inlining
#[inline]
fn small_fn(x: i32) -> i32 { x + 1 }
// Call sites replaced with x + 1

// Loop unrolling
for i in 0..4 {
    process(i); // Unrolled to 4 direct calls
}

// Vectorization (SIMD)
let sum: i32 = array.iter().sum(); // Uses SIMD instructions
```

Profile-guided optimization:
```bash
# Generate profile data
RUSTFLAGS="-Cprofile-generate" cargo build --release
./target/release/app  # Run typical workload
# Rebuild with profile
RUSTFLAGS="-Cprofile-use=default.profdata" cargo build --release
```

LLVM transforms Rust's MIR into optimized native code, making abstractions free.

## Connections
← [[022_zero_cost_abstractions]]
→ [[131_mir]]
→ [[132_pgo]]
→ [[133_simd]]

---
Level: L6
Date: 2025-08-15
Tags: #llvm #optimization #compiler #performance