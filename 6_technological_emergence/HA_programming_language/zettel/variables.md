# Variables

## Core Insight
Variables are names we give to memory locations - they're humanity's compromise between thinking in concepts and computing in addresses.

The word "variable" is a lie. In math, variables vary. In programming, they're often fixed:

```python
x = 5        # x "varies" to 5
x = x + 1    # x "varies" to 6
```

But what's really happening? We're not changing 5 into 6. We're changing which number the name 'x' refers to. The variable is the name, not the value.

In early computing, programmers thought in memory addresses:
```assembly
MOV [0x1234], 5    ; Store 5 at address 0x1234
ADD [0x1234], 1    ; Add 1 to value at 0x1234
```

Variables liberated us from this tyranny. Instead of remembering where data lives, we remember what it means:

```python
user_age = 25
account_balance = 1000.00
is_logged_in = True
```

But variables brought new problems:
- **Naming**: What to call things?
- **Scope**: Where do names have meaning?
- **Lifetime**: When does the memory get reclaimed?
- **Aliasing**: When do two names refer to the same thing?

Some languages rebelled against variable mutation:

```haskell
-- In Haskell, "variables" don't vary
let x = 5
let y = x + 1  -- x is still 5, forever
```

These aren't really variables - they're named constants. Once bound, always bound.

The deep insight: variables are the primary abstraction that makes programming accessible to humans. Without them, we're just very slow, error-prone addressing machines.

## Connections
→ [[naming_things]]
→ [[binding]]
→ [[memory_model]]
← [[storing_values]]

---
Level: L2
Date: 2025-06-22
Tags: #variables #names #memory #abstraction