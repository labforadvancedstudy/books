# Computability (L9)

## The Limits of Computation

Computability theory explores the fundamental question: What can and cannot be computed? It's the mathematical investigation of the boundaries of algorithmic problem-solving, revealing both the power and limitations of computation itself.

## The Church-Turing Thesis

The profound claim that anything computable can be computed by a Turing machine. This means:
- All reasonable models of computation are equivalent
- There's a universal notion of "algorithm"
- Some problems are absolutely unsolvable

## The Hierarchy of Problems

1. **Decidable**: Can be solved by an algorithm
2. **Semi-decidable**: Can find "yes" answers but might loop forever on "no"
3. **Undecidable**: No algorithm exists
4. **Uncomputable**: Beyond even theoretical computation

## The Halting Problem

The most famous undecidable problem:

```python
def halts(program, input):
    """
    Returns True if program halts on input,
    False if it loops forever.
    
    THEOREM: This function cannot exist.
    """
    # Proof by contradiction
    def paradox():
        if halts(paradox, None):
            while True: pass  # Loop forever
        else:
            return  # Halt
    
    # What does halts(paradox, None) return?
    # If True: paradox loops (contradiction)
    # If False: paradox halts (contradiction)
```

## Other Undecidable Problems

- **Post Correspondence**: Matching string sequences
- **Tiling Problem**: Can shapes tile the plane?
- **Diophantine Equations**: Integer solutions exist?
- **Program Equivalence**: Do two programs compute the same function?
- **Virus Detection**: Is a program malicious?

## Gödel's Incompleteness

The connection between computability and mathematical truth:
- Some true statements cannot be proven
- No formal system can prove its own consistency
- Mathematics itself has computational limits

## Computational Models

All equally powerful:
1. **Turing Machines**: Tape and state machine
2. **Lambda Calculus**: Function abstraction
3. **Register Machines**: Memory and instructions
4. **Cellular Automata**: Local rules, global behavior
5. **Quantum Turing Machines**: Quantum superposition

## The Computational Universe

```python
# Everything is computation?
class Universe:
    def __init__(self):
        self.state = initial_conditions()
    
    def step(self):
        # Apply laws of physics
        self.state = physics_rules(self.state)
    
    def run(self):
        while True:
            self.step()
            
# Are we inside such a loop?
```

## Practical Implications

Even decidable problems can be intractable:
- **P vs NP**: Can we verify faster than we can solve?
- **Exponential Time**: Technically solvable, practically impossible
- **PSPACE**: Problems requiring massive memory
- **BQP**: What quantum computers can efficiently solve

## Oracle Machines

What if we had magical help?

```python
class OracleMachine:
    def __init__(self, oracle):
        self.oracle = oracle  # Solves specific problem instantly
    
    def compute_with_oracle(self, input):
        # Even with oracle, some problems remain unsolvable
        # Leads to hierarchy of unsolvability
```

## The Philosophy of Limits

Computability theory reveals:
- **Absolute Limits**: Some questions have no answers
- **Relative Limits**: Some problems are harder than others
- **Physical Limits**: Reality constrains computation
- **Logical Limits**: Self-reference creates paradoxes

## Beyond Classical Computability

New models challenge classical limits:
- **Hypercomputation**: Computing beyond Turing
- **Quantum Computing**: Different complexity classes
- **Analog Computing**: Continuous vs discrete
- **Biological Computing**: DNA and proteins
- **Universe as Computer**: Is physics computable?

## The Existential Questions

- Is human consciousness computable?
- Are there thoughts we cannot think?
- Is free will compatible with computability?
- Could we simulate a universe identical to ours?
- Are we ourselves computable entities?

## Connection to Reality

If the universe is computational:
- Physical laws are algorithms
- Particles are data structures
- Time is iteration
- Space is memory
- We are subroutines

## Connection to Other Concepts

- **Algorithm** (L1): What computability studies
- **Halting Problem** (L8): Core undecidable problem
- **P vs NP** (L7): Complexity within computability
- **Quantum Algorithm** (L8): Extending computability?