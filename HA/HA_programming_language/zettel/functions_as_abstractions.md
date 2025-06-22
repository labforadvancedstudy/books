# Functions as Abstractions

## Core Insight
Functions are reusable thoughts - they package a computation into a name, letting us think at a higher level by hiding details we've already solved.

Before functions, programming was like writing a novel where you had to describe how to tie shoes every time a character put on shoes. Functions let us just say "tie_shoes()" and move on with the story.

```python
def calculate_area(radius):
    return 3.14159 * radius ** 2
```

Look what we've done. We've taken the concept "calculate the area of a circle" and crystallized it into a reusable unit. Now everywhere in our program, we can just think "calculate_area" instead of "multiply pi by radius squared."

But functions are more than convenience - they're abstraction barriers. The caller doesn't need to know HOW area is calculated, just THAT it is calculated. We could change the implementation to use a more precise pi, or a faster algorithm, and no calling code would break.

This is the fundamental trade of programming: we trade computer memory for human memory. The computer remembers the details so we don't have to.

Functions compose. Small functions build bigger functions build entire programs. It's abstraction all the way up:

```
pixels → draw_line() → draw_shape() → render_scene() → video_game
```

The profound insight: functions let us program in terms of WHAT, not HOW. They're the primary tool for managing complexity by hiding it.

## Connections
→ [[abstraction]]
→ [[composition_patterns]]
→ [[higher_order_functions]]
← [[naming_things]]

---
Level: L4
Date: 2025-06-22
Tags: #functions #abstraction #composition #modularity