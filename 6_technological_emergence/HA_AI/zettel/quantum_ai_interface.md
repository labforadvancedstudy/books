# Quantum AI Interface
<!-- Evidence Tier: Textbook (physics); Speculative (AGI framing) -->

## Core Insight
Where quantum computing meets AI — not "faster calculation" in the trivial sense, but a different resource (entanglement + interference) that yields provable speedups on *specific* problem classes.

> **[Correction 2026-04 — Pop-science quantum myth]** Previous text said: *"Quantum AI exists in superposition — considering multiple solutions simultaneously until measurement collapses to answer. It's not just parallel processing but quantum parallelism — being in all states until forced to choose."* This is a **widespread misconception** that Scott Aaronson, John Preskill, and essentially every working quantum information theorist have spent decades correcting.
>
> **What superposition actually is:** A quantum state is a single, deterministic vector in a complex Hilbert space. It is **not** the system being "in all states simultaneously." Measurement yields **one** classical outcome with probability |amplitude|². If a quantum computer evaluated all 2^N possibilities in parallel and let you read them out, we could solve NP-complete problems trivially — we **cannot**.
>
> **What actually gives speedup:** Interference between amplitudes arranged by a clever algorithm (Shor, Grover, HHL, quantum phase estimation) so that **wrong answers cancel** and right answers reinforce. Grover gives only √N speedup, not N. Shor factorizes in polynomial time but doesn't solve SAT.
>
> See Aaronson, "Quantum Computing Since Democritus" (2013); Nielsen & Chuang, *Quantum Computation and Quantum Information* (2010), §1.4.2 on the "exponential parallelism" fallacy.
>
> Evidence Tier: [Textbook]. The "being in all states until forced to choose" framing is rejected by mainstream quantum information theory.

**Honest statement:** Classical AI explores search spaces sequentially (or in parallel across many CPUs). Quantum algorithms on fault-tolerant hardware can, for *some* structured problems (factoring, certain linear systems, some sampling problems), achieve polynomial-to-exponential speedups — **not by "trying everything at once"** but by engineering amplitude interference.

**Near-term reality (2024–2026):** NISQ devices are noisy; variational quantum circuits for ML (QML) have not yet shown advantage over classical methods on practical benchmarks. Early experiments exist (quantum neural networks, quantum optimization, quantum sampling) but **no demonstrated ML advantage** on any real-world dataset as of 2024.

The "quantum nature might match fundamental uncertainty of intelligence" framing is **[Speculative]**, not a physics result.

## Connections
→ [[quantum_supremacy]]
→ [[superposition_processing]]
← [[quantum_machine_learning]]
← [[fundamental_computation]]

---
Level: L7
Date: 2025-06-21
Tags: #quantum #computation #superposition #fundamental