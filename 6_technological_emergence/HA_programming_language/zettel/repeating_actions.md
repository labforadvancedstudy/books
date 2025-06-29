# Repeating Actions

## Core Insight
Loops are humanity's gift to computers - teaching silicon to do what it does best: the same thing, over and over, without getting bored or making mistakes.

Humans hate repetition. "Write 'I will not talk in class' 100 times" is punishment. But computers? They live for it. They can repeat a billion times without fatigue, without variation, without complaint.

```
for i in range(1000000):
    calculate_something()
```

One million calculations. A human would go mad. A computer does it in milliseconds and asks "what's next?"

But loops are more than just repetition - they're abstraction over counting. Instead of writing:
```
print(students[0])
print(students[1])
print(students[2])
# ... oh god, there are 500 students
```

We write:
```
for student in students:
    print(student)
```

The loop doesn't care if there are 3 students or 3 million. It abstracts away the counting, the indexing, the boundary checking. It says: "do this thing for each thing in this collection of things."

The profound realization: loops are how we teach computers induction. "If this works for n, and this works for n+1, then it works for all n." It's mathematical induction made mechanical.

## Connections
→ [[control_flow]]
→ [[iteration_patterns]]
→ [[recursion]]
← [[sequence_of_instructions]]

---
Level: L2
Date: 2025-06-22
Tags: #loops #iteration #repetition #control-flow