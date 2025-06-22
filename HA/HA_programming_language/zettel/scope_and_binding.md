# Scope and Binding

## Core Insight
Scope is the universe where a name has meaning - it's how we create local realities within our programs where variables can exist without conflicting with the outside world.

Imagine if every variable ever created had to have a globally unique name. The millionth programmer to use 'x' would need to call it 'x_999999'. Madness. Scope saves us from this namespace apocalypse.

```python
def calculate_tax(income):
    rate = 0.3  # This 'rate' exists only here
    return income * rate

def calculate_interest(principal):
    rate = 0.05  # Different 'rate', different universe
    return principal * rate
```

Two functions, two rates, no conflict. Each function is its own universe with its own laws (variable bindings).

But scope is more than convenience - it's about controlling information flow. Variables in inner scopes can see outer scopes (usually), but not vice versa:

```python
x = 10  # Global scope
def foo():
    y = 20  # Local to foo
    def bar():
        z = x + y  # Can see both x and y
    # Can't see z here
# Can't see y or z here
```

This creates a hierarchy of knowledge. Inner functions know more than outer functions. It's information hiding in action.

The deep insight: scope is how we teach computers the concept of context. Just as "bank" means different things near a river versus near money, variables mean different things in different scopes.

## Connections
→ [[closures]]
→ [[lexical_vs_dynamic]]
→ [[environment_model]]
← [[naming_things]]

---
Level: L4
Date: 2025-06-22
Tags: #scope #binding #namespaces #context