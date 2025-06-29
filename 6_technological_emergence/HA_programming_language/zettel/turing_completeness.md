# Turing Completeness

## Core Insight
A system is Turing complete if it can compute anything computable - it's the universe of computation's minimum viable feature set.

What does it take to compute everything computable? Surprisingly little:
- A way to store data (memory)
- A way to make decisions (conditionals)
- A way to repeat (loops or recursion)

That's it. With these three ingredients, you can compute anything that can be computed. Your language might be painful to use, but it's theoretically as powerful as any supercomputer.

Accidentally Turing complete systems are everywhere:
- CSS + HTML (with checkbox hacks)
- PowerPoint animations
- Minecraft redstone
- Magic: The Gathering
- Conway's Game of Life

People keep discovering that systems designed for one purpose can compute anything. It's like finding out your toaster can solve differential equations.

```python
# The smallest Turing complete language?
# Just these operations:
# - Increment data pointer: >
# - Decrement data pointer: <  
# - Increment byte at pointer: +
# - Decrement byte at pointer: -
# - Output byte at pointer: .
# - Input byte to pointer: ,
# - Jump forward if zero: [
# - Jump backward if nonzero: ]

# This is Brainfuck, and it can compute anything
```

The profound implication: there's a computational bedrock. Once you hit Turing completeness, adding features makes programming easier, not more powerful. C and Haskell and Prolog can all compute exactly the same set of functions.

But there's a dark side: the halting problem. Turing completeness brings undecidability. You can't analyze these systems completely. You can't optimize them perfectly. Power comes with fundamental limits.

## Connections
→ [[church_turing_thesis]]
→ [[universal_computation]]
→ [[halting_problem]]
← [[computation_models]]

---
Level: L6
Date: 2025-06-22
Tags: #turing #completeness #computation #universality