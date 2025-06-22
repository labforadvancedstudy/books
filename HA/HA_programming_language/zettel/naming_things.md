# Naming Things

## Core Insight
Naming is the fundamental act of programming - it's how we teach the machine our mental model of the problem.

"There are only two hard things in Computer Science: cache invalidation and naming things." - Phil Karlton

Why is naming so hard? Because a name must bridge two worlds: the human understanding and the machine execution. A good name tells a story. `calculateTotalPrice` whispers its purpose. `x` stays mute.

In the beginning, programmers used single letters. Memory was precious, keyboards were hard. But as programs grew, we learned: code is read far more often than written. Names are documentation that can't go out of sync.

Consider these evolutions:
```
i → index → currentIndex → currentStudentIndex
tmp → temp → temporary → temporaryBuffer → unsavedUserData
```

Each rename adds context, reduces cognitive load. The computer doesn't care - `a` or `averageMonthlyTemperature` compile to the same thing. But for the human mind parsing thousands of lines? Names are everything.

Some cultures believe knowing something's true name gives power over it. Programmers live this daily. The right name makes bugs obvious, makes code intentions clear, makes maintenance possible.

## Connections
→ [[variables]]
→ [[scope_and_binding]]
→ [[abstraction]]
← [[symbols_and_meaning]]

---
Level: L2
Date: 2025-06-22
Tags: #naming #abstraction #readability #cognitive-load