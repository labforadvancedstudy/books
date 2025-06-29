# Lambda Calculus

## Core Insight
Lambda calculus strips computation to its absolute essence: everything is a function, even numbers, even booleans, even functions themselves.

Alonzo Church asked: what if we had only functions? No numbers, no data structures, no loops, no anything. Just functions. Could we still compute?

The lambda calculus has exactly three constructs:
1. **Variables**: x, y, z
2. **Abstraction**: λx.e (a function with parameter x and body e)
3. **Application**: (f a) (apply function f to argument a)

That's it. The entire language. Yet from this minimalism springs all of computation.

How do we represent numbers without numbers?

```
0 = λf.λx.x                    # Apply f zero times to x
1 = λf.λx.f x                  # Apply f once to x
2 = λf.λx.f (f x)              # Apply f twice to x
3 = λf.λx.f (f (f x))          # Apply f three times to x
```

These are Church numerals. Numbers are functions that apply a function n times.

How about booleans?

```
TRUE  = λx.λy.x                # Takes two arguments, returns first
FALSE = λx.λy.y                # Takes two arguments, returns second
IF = λp.λa.λb.p a b            # Pass both branches to the predicate
```

Even recursion, without any built-in recursion:

```
Y = λf.(λx.f (x x))(λx.f (x x))  # The Y combinator
```

This function, when applied to another function, creates a recursive version of it. Recursion from nothing but functions.

The stunning realization: every programming language is just lambda calculus with syntactic sugar. Strip away the keywords, the types, the syntax, and you'll find lambdas all the way down.

## Connections
→ [[church_encodings]]
→ [[beta_reduction]]
→ [[combinatory_logic]]
← [[computation_models]]

---
Level: L7
Date: 2025-06-22
Tags: #lambda-calculus #foundations #functions #church