# Storing Values

## Core Insight
Memory is just organized forgetfulness - we store values not because they're permanent, but because we need them to persist just long enough to be useful.

Before computers, humans stored values in their heads, on their fingers, in sand, on paper. The act is ancient: making a mark that means "remember this." Programming simply mechanized this urge.

```
age = 25
```

What really happens here? Somewhere in silicon, electrons arrange themselves in a pattern. That pattern, through layers of abstraction, means "25". We give this pattern a name - "age" - so we can find it again.

But here's the magic: the computer doesn't know this is an age. It doesn't know what 25 means. It just knows: at memory location 0x7ffe5694a3d0, store this bit pattern: 00011001.

Variables are humanity's compromise with computer memory. Instead of remembering "the value at address 0x7ffe5694a3d0", we say "age". Instead of thinking in bits, we think in human concepts.

The deepest insight: all of computing is just very organized ways of storing and retrieving values. Everything else - functions, objects, programs themselves - are just patterns of stored values, interpreted in specific ways.

## Connections
→ [[variables]]
→ [[memory_model]]
→ [[types_and_values]]
← [[symbols_and_meaning]]

---
Level: L1
Date: 2025-06-22
Tags: #memory #storage #variables #state