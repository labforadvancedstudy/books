# Sequence of Instructions

## Core Insight
Programming is fundamentally about time - instructing a machine to do this, then that, then the other, creating a dance of state changes through sequential steps.

A recipe says: first crack the eggs, then whisk them, then pour into the pan. The order matters. You can't pour uncracked eggs. This is so obvious to humans we barely think about it, but it's the foundation of all programming.

The computer, beautiful idiot that it is, does exactly what you tell it, in exactly the order you tell it:

```
x = 5      // First, x becomes 5
y = x + 3  // Then, y becomes 8  
x = 2      // Now x becomes 2, but y is still 8
```

Swap any two lines, the meaning changes. This rigid sequentiality is both programming's greatest strength and its most common source of bugs. We think in goals ("make coffee"), computers think in steps ("1. Get mug. 2. Add coffee grounds. 3...").

The profound realization: all computation is just matter rearranging itself in time. Each instruction changes the state of the machine slightly, and millions of these tiny changes per second create the illusion of intelligence.

Even parallel programming doesn't escape sequence - it just runs multiple sequences at once, introducing a whole new hell of coordination problems.

## Connections
→ [[control_flow]]
→ [[state_and_mutation]]
→ [[evaluation_order]]
← [[instruction_set]]

---
Level: L1
Date: 2025-06-22
Tags: #sequence #instructions #imperative #time #order