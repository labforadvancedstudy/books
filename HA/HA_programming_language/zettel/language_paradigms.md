# Language Paradigms

## Core Insight
A paradigm isn't just syntax - it's a worldview that shapes how we decompose problems, structure solutions, and think about computation itself.

Imperative programming sees the world as a recipe: do this, then that. It mirrors how we experience time - sequential, stateful, cause-and-effect.

```c
int sum = 0;
for(int i = 0; i < n; i++) {
    sum += array[i];
}
```

Functional programming sees the world as mathematical transformations: input becomes output through pure functions. No time, no state, just mappings.

```haskell
sum = fold (+) 0 array
```

Object-oriented programming sees the world as interacting entities: objects with state and behavior, sending messages, hiding internals.

```java
class BankAccount {
    private double balance;
    public void deposit(double amount) { ... }
}
```

Logic programming sees the world as facts and rules: state what's true, let the computer figure out how to make it so.

```prolog
parent(tom, bob).
parent(bob, ann).
grandparent(X, Z) :- parent(X, Y), parent(Y, Z).
```

Each paradigm isn't just a different syntax - it's a different metaphysics. OOP programmers think in nouns (objects). Functional programmers think in verbs (transformations). Logic programmers think in relationships.

The revolution: multi-paradigm languages. Because reality doesn't fit in one paradigm. Some problems scream for objects, others for functions, others for rules. Modern languages let us switch worldviews as needed.

## Connections
→ [[functional_programming]]
→ [[object_oriented]]
→ [[declarative_vs_imperative]]
← [[computation_models]]

---
Level: L6
Date: 2025-06-22
Tags: #paradigms #programming-styles #worldview #design