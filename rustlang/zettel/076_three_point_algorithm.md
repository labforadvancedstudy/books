# Three Point Algorithm

## Core Insight
The three-point algorithm tracks where borrows are created, used, and could be invalidated, enabling NLL's precise lifetime analysis.

The three points:
1. **Creation point**: Where borrow is created
2. **Use points**: Where borrow is used
3. **Invalidation points**: Where borrow would become invalid

```rust
let mut vec = vec![1, 2, 3];
let r = &vec[0];      // Point 1: Creation
println!("{}", r);     // Point 2: Use
                      // Point 3: Last use (NLL ends borrow here)
vec.push(4);          // OK with NLL, error without
```

Algorithm process:
```rust
// Compiler tracks:
{
    let mut data = String::new();
    let r1 = &data;        // Region 'r1 starts
    process(r1);           // 'r1 used
    let r2 = &mut data;    // Would invalidate 'r1
    // But 'r1 not used after process(), so OK
    modify(r2);            // 'r2 used
}
```

Benefits:
- More precise than lexical scopes
- Fewer false positive errors
- Natural code patterns work
- Better error messages

The algorithm performs dataflow analysis to compute minimal borrow lifetimes.

## Connections
← [[028_nll]]
← [[006_borrow_checker]]
→ [[147_region_inference]]

---
Level: L6
Date: 2025-08-15
Tags: #nll #algorithm #borrow-checker #dataflow