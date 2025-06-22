# Control Flow

## Core Insight
Control flow is the choreography of computation - the patterns we use to direct the dance of execution through our programs.

Programs aren't just sequences. They're branching, looping, jumping mazes of possibility. Control flow structures are how we tame this complexity into understandable patterns.

The basic moves:
- **Sequence**: First this, then that
- **Selection**: If this, then that, else the other
- **Iteration**: While this, do that
- **Recursion**: To do this, do this with a simpler version

But these simple patterns combine into complex choreographies:

```python
for user in users:
    if user.is_active():
        for order in user.orders:
            if order.is_pending():
                process_order(order)
            else:
                archive_order(order)
```

Nested loops and conditions create a decision tree, each path a different dance through the code.

Then come the rule-breakers:
- **Break**: Exit the dance early
- **Continue**: Skip to the next beat
- **Goto**: Teleport anywhere (considered harmful!)
- **Exceptions**: When the music stops unexpectedly

Modern control flow gets exotic:
- **Generators**: Pause the dance, resume later
- **Async/await**: Multiple dancers, taking turns
- **Pattern matching**: Choose your dance by the shape of your partner

The deep insight: control flow structures are how we encode time and choice into static text. They're the difference between a recipe and a meal, between a score and a symphony.

## Connections
→ [[structured_programming]]
→ [[goto_considered_harmful]]
→ [[coroutines_and_continuations]]
← [[sequence_of_instructions]]

---
Level: L3
Date: 2025-06-22
Tags: #control-flow #structure #patterns #execution