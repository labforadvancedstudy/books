# Level 5: Composition and Transformation - The Algebra of Programs
*How simple pieces combine into complex behaviors*

> "The programmer, like the poet, works only slightly removed from pure thought-stuff." - Fred Brooks

## Functions as First-Class Citizens

In the beginning, functions were just subroutines - places to jump to. Then came a revolution: what if functions were values?

```python
# Functions as values
def shout(text):
    return text.upper() + "!"

def whisper(text):
    return text.lower() + "..."

# Store in variables
my_voice = shout
print(my_voice("hello"))  # HELLO!

# Put in lists
voices = [shout, whisper]
for voice in voices:
    print(voice("Hello"))  # HELLO! then hello...
```

This changes everything. Functions become things you can pass around, store, and manipulate. The boundary between code and data blurs.

## Higher-Order Thinking

Functions that eat functions and birth new functions:

```python
def make_repeater(n):
    def repeat(f):
        def repeated(x):
            result = x
            for _ in range(n):
                result = f(result)
            return result
        return repeated
    return repeat

triple = make_repeater(3)
triple_shout = triple(shout)
print(triple_shout("hi"))  # HI!!!
```

We're not just transforming data anymore. We're transforming transformations. This is meta-programming at its purest.

## The Holy Trinity: Map, Filter, Reduce

Three patterns that capture 90% of list processing:

```python
numbers = [1, 2, 3, 4, 5]

# Map: transform each
squared = list(map(lambda x: x**2, numbers))  # [1, 4, 9, 16, 25]

# Filter: select some  
evens = list(filter(lambda x: x % 2 == 0, numbers))  # [2, 4]

# Reduce: combine all
from functools import reduce
total = reduce(lambda a, b: a + b, numbers)  # 15
```

These aren't just functions - they're a different way of thinking. Instead of telling the computer HOW to loop, we tell it WHAT transformation we want.

## Lazy vs Eager: When to Compute

Eager evaluation computes now:
```python
def eager_list():
    print("Computing all values...")
    return [expensive_calculation(x) for x in range(1000000)]

results = eager_list()  # Computes everything immediately
first = results[0]      # Just accessing pre-computed value
```

Lazy evaluation computes later:
```python
def lazy_generator():
    print("Creating generator...")
    for x in range(1000000):
        yield expensive_calculation(x)

results = lazy_generator()  # Creates generator instantly
first = next(results)       # NOW it computes the first value
```

Laziness lets us work with infinite sequences:
```python
def fibonacci():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

# Take just what we need from infinity
for i, num in enumerate(fibonacci()):
    if i >= 10:
        break
    print(num)  # 0, 1, 1, 2, 3, 5, 8, 13, 21, 34
```

## Side Effects: Where Purity Meets Reality

Pure functions live in Plato's realm:
```python
# Pure: no side effects
def add(a, b):
    return a + b

# Side effects everywhere
def messy_add(a, b):
    print(f"Adding {a} and {b}")      # I/O effect
    global call_count
    call_count += 1                    # State mutation
    log_to_file(f"{a} + {b}")         # File system effect
    if random.random() < 0.1:          # Non-determinism
        raise Exception("Random fail")  # Exception effect
    return a + b
```

Side effects are where programs touch the world. Without them, computation is just expensive heating. With them, programs become unpredictable.

## Composition Patterns

Building complex behavior from simple pieces:

```python
# Pipeline pattern
def pipeline(*functions):
    def piped(value):
        for func in functions:
            value = func(value)
        return value
    return piped

process = pipeline(
    str.strip,
    str.lower,
    lambda s: s.replace(' ', '_'),
    lambda s: f"user_{s}"
)

print(process("  John Doe  "))  # user_john_doe
```

Composition is algebra for functions:
- Identity: `f(x) = x`
- Associative: `f(g(h(x))) = (f∘g)(h(x)) = f((g∘h)(x))`
- Not commutative: `f(g(x)) ≠ g(f(x))` usually

## Currying: One Argument at a Time

Breaking multi-argument functions into chains:

```python
# Normal function
def add(a, b):
    return a + b

# Curried version
def curried_add(a):
    def add_a(b):
        return a + b
    return add_a

add_five = curried_add(5)
print(add_five(3))  # 8

# More elegant with lambdas
curry = lambda f: lambda a: lambda b: f(a, b)
curried_multiply = curry(lambda x, y: x * y)
double = curried_multiply(2)
```

Currying lets us partially apply functions, creating specialized versions. It's how we build vocabulary from primitives.

## Monads: Wrapping Context

The scariest word in functional programming, but just a pattern for handling context:

```python
# Maybe monad - handling potential None
class Maybe:
    def __init__(self, value):
        self.value = value
    
    def bind(self, func):
        if self.value is None:
            return Maybe(None)
        return Maybe(func(self.value))
    
    def __repr__(self):
        return f"Maybe({self.value})"

# Chain operations that might fail
result = (Maybe(10)
    .bind(lambda x: x * 2)
    .bind(lambda x: x + 5)
    .bind(lambda x: x if x < 30 else None)
    .bind(lambda x: x * 10))

print(result)  # Maybe(None) - because 25 < 30 is False
```

Monads let us compose operations while handling failure, async, state, or any other context uniformly.

## Streams and Infinite Data

Working with data that never ends:

```python
import itertools

# Infinite sequence of natural numbers
naturals = itertools.count(1)

# Transform infinitely
squares = map(lambda x: x**2, naturals)

# Filter infinitely  
even_squares = filter(lambda x: x % 2 == 0, squares)

# Take finite slice from infinity
first_10 = list(itertools.islice(even_squares, 10))
print(first_10)  # [4, 16, 36, 64, 100, 144, 196, 256, 324, 400]
```

Streams let us describe infinite computations finitely. We separate the description of what to compute from the decision of how much to compute.

## The Expression Problem

How do we add new operations to existing types? New types to existing operations?

```python
# Object-oriented: easy to add types, hard to add operations
class Shape:
    def area(self): pass

class Circle(Shape):
    def area(self): return pi * self.r**2

class Square(Shape):
    def area(self): return self.side**2

# Functional: easy to add operations, hard to add types
def area(shape):
    if shape['type'] == 'circle':
        return pi * shape['r']**2
    elif shape['type'] == 'square':
        return shape['side']**2

def perimeter(shape):
    if shape['type'] == 'circle':
        return 2 * pi * shape['r']
    elif shape['type'] == 'square':
        return 4 * shape['side']
```

Neither paradigm solves this perfectly. It's a fundamental tension in language design.

## The Real Mystery Is...

Why does composition work so well?

In the physical world, putting two things together often creates emergent chaos. But in the computational world, composing two simple functions reliably creates a more complex but still understandable function.

This shouldn't work as well as it does. When we pipeline 10 functions together, why doesn't the complexity explode? When we map a function over a nested data structure, why does it just work?

The answer might be that computation is fundamentally about information transformation, and information composes more cleanly than matter. When we transform data, we're working with pure structure, free from the messy contingencies of the physical world.

Or maybe we've just gotten very good at designing languages and abstractions that make composition work. We've learned to build our functions like LEGO blocks - with standardized interfaces that snap together predictably.

Whatever the reason, composition is our superpower. It lets us build programs too complex for any one person to understand, yet that still work. We stand on the shoulders of functions, all the way down.

---

*"Make it work, make it right, make it fast." - Kent Beck*

*Next: [Level 6 - Models of Computation →](L6_Models_of_Computation.md)*