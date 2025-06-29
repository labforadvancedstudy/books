# Side Effects

## Core Insight
Side effects are when functions do more than return values - they change the world beyond their scope, making programs powerful but unpredictable.

In mathematics, sin(π) = 0. Always. Forever. Calling sin doesn't change π, doesn't change sin, doesn't change anything except producing a result.

In programming:

```python
def increment_counter():
    global counter
    counter += 1      # Side effect: changes global state
    print(counter)    # Side effect: I/O to console
    log_to_file()     # Side effect: writes to disk
    return counter    # The "main" effect
```

This function doesn't just compute - it mutates state, performs I/O, potentially crashes, takes time, uses memory. It has effects beyond returning a value.

Side effects are everywhere:
- Modifying variables
- Reading/writing files
- Network requests
- User input/output
- Throwing exceptions
- Even allocating memory

Pure functional languages say: "Side effects are evil. Ban them!"
But wait - a program with no side effects does nothing observable. It might as well not run.

The compromise: controlled side effects. Isolate them. Mark them. Track them.

```haskell
-- Haskell marks effects in the type system
getLine :: IO String          -- This function does I/O
length :: String -> Int        -- This function is pure

-- You can't hide effects
sneaky :: String -> Int
sneaky s = length (getLine)   -- Type error! Can't hide IO
```

The profound realization: side effects are where programs touch reality. Pure functions live in Plato's realm of ideals. Side effects drag them into the messy physical world of time, space, and change.

## Connections
→ [[pure_functions]]
→ [[referential_transparency]]
→ [[effect_systems]]
← [[state_and_mutation]]

---
Level: L5
Date: 2025-06-22
Tags: #side-effects #purity #functional #state