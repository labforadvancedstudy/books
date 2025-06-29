# Types and Values

## Core Insight
Types are promises about what values can do - they're contracts that let us reason about programs without running them.

In the beginning, there were just bits. Patterns of 1s and 0s. But 01000001 - is that 65? The letter 'A'? A processor instruction? Part of a floating-point number? Without context, bits mean nothing.

Types are that context. They say: "these bits represent an integer" or "these bits are text" or "these bits are a function waiting to be called."

```python
age = 25          # int type
name = "Alice"    # string type  
height = 5.9      # float type
```

But types go deeper than labeling. They're about capabilities:
- Integers can be added, subtracted, compared
- Strings can be concatenated, sliced, searched
- Functions can be called, passed around, composed

Static typing (Java, Rust) says: "Prove to me at compile time that this will work."
Dynamic typing (Python, JavaScript) says: "Trust me, I'll check when we get there."

The holy war between them misses the point: both are trying to prevent the fundamental horror of programming - using a value in a way it wasn't meant to be used. Trying to uppercase a number. Trying to add a function to a string.

Types are humanity's attempt to classify the infinite variety of data into manageable categories. It's taxonomy for the digital world.

## Connections
→ [[type_systems]]
→ [[type_inference]]
→ [[polymorphism]]
← [[storing_values]]

---
Level: L3
Date: 2025-06-22
Tags: #types #values #type-systems #semantics