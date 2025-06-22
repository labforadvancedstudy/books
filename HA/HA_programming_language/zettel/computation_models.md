# Computation Models

## Core Insight
A computation model is a universe with its own laws of physics - defining what can be computed, how it's computed, and what computation even means.

Turing gave us the tape machine - infinite tape, read/write head, simple rules. Church gave us lambda calculus - functions all the way down. Von Neumann gave us stored programs - code and data in the same memory. Each model is a different answer to "What is computation?"

The Turing Machine says: computation is state transitions. Start in state A, read symbol, write symbol, move, go to state B. It's mechanical, sequential, almost physical.

Lambda calculus says: computation is function application. (λx.x+1) 5 → 6. No state, no sequence, just transformation. Pure and mathematical.

The von Neumann architecture says: computation is fetch-decode-execute. Get instruction from memory, figure out what it means, do it, repeat. It's the cycle that runs billions of times per second in your computer right now.

But here's the miracle: Church-Turing thesis says these are all equivalent! Any computation possible in one model is possible in all others. They're different lenses for viewing the same fundamental phenomenon.

Modern languages mix models. JavaScript has von Neumann-style statements AND lambda calculus functions. Haskell looks like pure lambda calculus but runs on von Neumann hardware. The models blend and blur.

The deep question: Is human thought also computation? If so, which model? Or do we need a new model entirely?

## Connections
→ [[turing_completeness]]
→ [[lambda_calculus]]
→ [[automata_theory]]
← [[language_paradigms]]

---
Level: L6
Date: 2025-06-22
Tags: #computation #models #theory #foundations