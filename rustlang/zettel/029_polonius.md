# Polonius

## Core Insight
Polonius is the next-generation borrow checker using datalog and differential dataflow for more precise lifetime analysis.

Current borrow checker limitations:
```rust
fn get_or_insert<'a>(map: &'a mut HashMap<u32, String>) -> &'a String {
    match map.get(&42) {
        Some(s) => s, // ERROR: can't return immutable ref while map borrowed mutably
        None => {
            map.insert(42, String::from("default"));
            &map[&42]
        }
    }
}
```

Polonius would understand:
- The immutable borrow from `get()` doesn't overlap with `insert()`
- More precise "liveness" tracking
- Conditional borrowing patterns
- Better handling of loops and control flow

Key improvements:
- Location-sensitive analysis (not just lexical)
- Supports "problem cases" from RFC 2094
- Uses logical rules (datalog) instead of imperative algorithm
- Incremental computation via differential dataflow

Status: Experimental, can be enabled with `-Z polonius` flag. Will eventually replace current borrow checker.

## Connections
← [[006_borrow_checker]]
← [[028_nll]]
→ [[077_datalog_analysis]]

---
Level: L6
Date: 2025-08-15
Tags: #polonius #borrow-checker #compiler #experimental