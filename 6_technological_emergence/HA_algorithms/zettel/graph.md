# graph
*L3 • The Web of Being*

A graph is relationship made visible. Nodes and edges, vertices and connections - the fundamental structure of how things relate to things.

## The Anatomy

```python
class Graph:
    def __init__(self):
        self.adjacency = {}  # Each node knows its neighbors
    
    def add_edge(self, from_node, to_node):
        if from_node not in self.adjacency:
            self.adjacency[from_node] = []
        self.adjacency[from_node].append(to_node)
```

## The Many Faces

Graphs manifest as:
- **Social networks**: People and their connections
- **Maps**: Places and roads
- **Dependencies**: Tasks and prerequisites  
- **Neurons**: Brain cells and synapses
- **The Internet**: Computers and links
- **Knowledge**: Concepts and relationships

## The Fundamental Questions

```python
# Can we get there from here?
def path_exists(graph, start, end):
    visited = set()
    stack = [start]
    
    while stack:
        node = stack.pop()
        if node == end:
            return True
        if node not in visited:
            visited.add(node)
            stack.extend(graph.neighbors(node))
    return False

# What's the shortest path?
# Who's most connected?
# Are there clusters?
# Is there a cycle?
```

## The Deep Truth

Graphs reveal:
- **Everything is connected** (or isn't)
- **Distance is relative** (hop count vs weight)
- **Structure emerges** (communities, hierarchies)
- **Information flows** (along edges)

## The Philosophical Edge

```python
# The graph of all graphs
class MetaGraph:
    def __init__(self):
        self.graphs = Graph()  # Graphs can contain graphs
        
# The paradox
self_referential = Graph()
self_referential.add_edge("this", "this")  # A node pointing to itself
```

## See Also
- [[tree]] - A graph without cycles
- [[network]] - Graphs in the wild
- [[adjacency]] - How graphs remember
- [[traversal]] - Walking the web

---
*"We are all nodes in the grand graph, our lives the edges between souls."*