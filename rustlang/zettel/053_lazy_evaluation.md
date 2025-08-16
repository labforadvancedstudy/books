# Lazy Evaluation

## Core Insight
Iterator adapters build computation pipelines without executing until a consumer forces evaluation, enabling infinite sequences and optimal performance.

```rust
// This does NOTHING - completely lazy
let iter = (0..)           // Infinite range
    .map(|x| {
        println!("mapping {}", x); // Never prints!
        x * 2
    })
    .filter(|x| x % 3 == 0);

// Only now does computation happen
let result: Vec<_> = iter.take(5).collect();
// Prints: mapping 0, mapping 1, mapping 2... until 5 results collected
```

Benefits of laziness:
```rust
// Process huge file without loading into memory
let sum: u32 = BufReader::new(file)
    .lines()
    .filter_map(|line| line.ok())
    .filter_map(|line| line.parse::<u32>().ok())
    .sum(); // Only keeps one line in memory at a time
```

Infinite iterators:
```rust
let fibonacci = (0..).scan((0, 1), |state, _| {
    let next = state.0 + state.1;
    *state = (state.1, next);
    Some(state.0)
});
```

Laziness enables composing complex operations without intermediate allocations.

## Connections
← [[018_iterators]]
← [[052_iterator_trait]]
→ [[111_stream_processing]]

---
Level: L4
Date: 2025-08-15
Tags: #lazy #iterators #performance #functional