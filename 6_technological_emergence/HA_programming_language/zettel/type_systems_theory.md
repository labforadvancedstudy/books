# Type Systems Theory

## Core Insight
Type systems are mathematical proofs about programs - they're formal logic applied to code, proving the absence of certain errors without running the program.

Types started simple: integers, strings, booleans. But type theory revealed something profound: types are propositions, programs are proofs. This is the Curry-Howard correspondence, one of the deepest insights in computer science.

Consider:
```haskell
-- A function type is an implication
f :: Int -> String
-- Reads as: "If you give me an Int, I can prove a String exists"

-- A product type is a conjunction  
(Int, String)
-- "I can prove both an Int AND a String exist"

-- A sum type is a disjunction
Either Int String  
-- "I can prove an Int OR a String exists"

-- An empty type is falsehood
Void
-- "I cannot prove this"
```

Suddenly, programming becomes logic. Type checking becomes theorem proving.

Modern type systems go wild with power:
- **Generics**: Types parameterized by types
- **Dependent types**: Types that depend on values
- **Linear types**: Types that track resource usage
- **Effect types**: Types that track side effects

```typescript
// TypeScript: Types computing types
type ElementType<T> = T extends (infer E)[] ? E : never;
type Num = ElementType<number[]>;  // number
```

The type system has become a programming language itself!

But there's a trade-off. More powerful type systems can prove more properties, but they're harder to use. At the extreme, dependently typed languages can prove almost anything - if you're willing to write the proof.

The philosophical question: How much should we prove? Total correctness requires total specification. But if the specification is as complex as the program, what have we gained?

## Connections
→ [[curry_howard_correspondence]]
→ [[dependent_types]]
→ [[type_inference]]
← [[types_and_values]]

---
Level: L7
Date: 2025-06-22
Tags: #types #theory #logic #proofs #formal-methods