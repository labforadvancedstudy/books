# Data Structure (L3)

## The Architecture of Information

A data structure is a way of organizing and storing data so that it can be accessed and modified efficiently. It's the scaffolding that gives shape to raw information, transforming chaos into order.

## The Fundamental Question

How do we arrange data to make it useful? This is the question every data structure answers, each in its own way.

## Basic Building Blocks

1. **Arrays**: Sequential, indexed storage
2. **Linked Lists**: Connected nodes of data
3. **Trees**: Hierarchical relationships
4. **Graphs**: Networks of connections
5. **Hash Tables**: Key-value associations

## The Trade-offs

Every data structure makes choices:
- **Space vs Time**: Use more memory for faster access?
- **Flexibility vs Efficiency**: General purpose or optimized?
- **Simplicity vs Power**: Easy to use or feature-rich?

## Real-World Analogies

- **Array**: Numbered mailboxes in an apartment building
- **Stack**: Pile of plates (last in, first out)
- **Queue**: Line at a coffee shop (first in, first out)
- **Tree**: Family genealogy
- **Graph**: Social network connections

## Code Examples

```python
# The simplest structure: a container
class Box:
    def __init__(self, item):
        self.item = item

# A fundamental structure: linked nodes
class Node:
    def __init__(self, data):
        self.data = data
        self.next = None

# Abstract data type: Stack
class Stack:
    def __init__(self):
        self.items = []
    
    def push(self, item):
        self.items.append(item)
    
    def pop(self):
        return self.items.pop()
```

## The Philosophy

Data structures are more than just containers - they encode relationships, impose constraints, and embody assumptions about how information will be used. They are the grammar of computation.

## Why They Matter

Choosing the right data structure can be the difference between a program that runs in seconds and one that takes hours. They are the foundation upon which efficient algorithms are built.

## Connection to Other Levels

- **Algorithm** (L1): What processes these structures
- **Optimization** (L4): Making structures more efficient
- **Tree** (L3): A specific hierarchical structure
- **Graph** (L3): The most general structure