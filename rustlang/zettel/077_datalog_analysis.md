# Datalog Analysis

## Core Insight
Polonius uses Datalog rules to express borrow checking as logical relations, enabling more precise and maintainable lifetime analysis.

Datalog representation:
```prolog
% Facts
loan_issued_at(L, P) :- /* loan L issued at point P */
access_path(V, P) :- /* variable V accessed at P */

% Rules
borrow_live_at(L, P) :-
    loan_issued_at(L, P0),
    access_path(V, P),
    P0 <= P.

error(P) :-
    borrow_live_at(L1, P),
    borrow_live_at(L2, P),
    conflicts(L1, L2).
```

Benefits over imperative algorithm:
- Declarative: what, not how
- Compositional: rules combine naturally
- Incremental: differential dataflow
- Verifiable: easier to prove correct

Example rule:
```rust
// "A borrow is live if used later"
// In Datalog:
// borrow_live_at(Borrow, Point) :-
//     borrow_used_at(Borrow, LaterPoint),
//     Point < LaterPoint.

let x = &mut data;  // Borrow created
// ... many lines ...
use_ref(x);         // Borrow used
// Datalog infers borrow live throughout
```

Datalog makes complex lifetime relationships expressible as simple logical rules.

## Connections
← [[029_polonius]]
→ [[148_differential_dataflow]]
→ [[149_logic_programming]]

---
Level: L7
Date: 2025-08-15
Tags: #datalog #polonius #logic #analysis