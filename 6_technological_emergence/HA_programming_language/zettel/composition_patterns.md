# Composition Patterns

## Core Insight
Composition is the art of building complexity from simplicity - creating rich behaviors by combining simple pieces in structured ways.

In the beginning, we had monoliths. Thousand-line functions that did everything. Then we learned: big things should be made of small things. But how do the small things fit together?

**Function composition**: The mathematician's dream.
```haskell
(f . g) x = f(g(x))
```
Simple, elegant, powerful. Build complex transformations by chaining simple ones.

**Object composition**: The engineer's approach.
```python
class Car:
    def __init__(self):
        self.engine = Engine()
        self.wheels = [Wheel() for _ in range(4)]
```
Build complex objects from simpler objects. Has-a instead of is-a.

**Pipeline composition**: The Unix philosophy.
```bash
cat file.txt | grep pattern | sort | uniq
```
Small tools, each doing one thing well, combined into powerful workflows.

But composition is more than mechanical assembly. It's about:
- **Interfaces**: How pieces fit together
- **Contracts**: What pieces promise to each other
- **Emergence**: The whole becoming more than its parts

The profound realization: all of programming is composition. Variables compose into expressions. Expressions compose into statements. Statements compose into functions. Functions compose into modules. Modules compose into programs. Programs compose into systems.

It's composition all the way down. And up.

## Connections
→ [[modularity]]
→ [[interfaces_and_contracts]]
→ [[emergence_in_systems]]
← [[functions_as_abstractions]]

---
Level: L5
Date: 2025-06-22
Tags: #composition #patterns #modularity #design