# Computational Limits

## Core Insight
There are problems no program can solve, questions no algorithm can answer - computation has fundamental, provable limits that humble our digital ambitions.

Turing's halting problem shattered the dream of perfect computation. Can we write a program that determines if any program halts or runs forever?

```python
def halts(program, input):
    # Returns True if program(input) halts
    # Returns False if it runs forever
    # ... but this function cannot exist!
```

The proof is diabolical. If `halts` existed, we could write:

```python
def paradox():
    if halts(paradox, None):
        while True: pass  # Loop forever
    else:
        return  # Halt
```

If `paradox` halts, it runs forever. If it runs forever, it halts. Contradiction. Therefore, `halts` cannot exist.

This isn't a bug. It's a feature of reality. There are limits to knowledge, even for perfect computers with infinite time and memory.

More limits pile up:
- **Gödel's incompleteness**: Some truths can't be proven
- **Rice's theorem**: Any non-trivial property of programs is undecidable
- **Complexity barriers**: Some problems take longer than the universe's lifetime

But here's the twist: these limits are liberating. They mean:
- Creativity can't be mechanized (fully)
- Humans aren't obsolete
- There will always be mysteries
- The universe can surprise us

Programming languages dance at these boundaries. Type systems try to prove program properties (but can't prove everything). Optimizers try to predict behavior (but can't predict everything).

The deepest question: Are human minds also bounded by these limits? Or do we compute in ways Turing never imagined?

## Connections
→ [[halting_problem]]
→ [[undecidability]]
→ [[complexity_theory]]
← [[computation_models]]

---
Level: L9
Date: 2025-06-22
Tags: #limits #computation #undecidability #philosophy