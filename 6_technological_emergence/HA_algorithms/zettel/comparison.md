# comparison
*L1 • The Fundamental Question*

Comparison is the atom of decision. Every algorithm, at its core, asks: is this greater than, less than, or equal to that?

## The Three-Fold Path

```python
# The trinity of comparison
def compare(a, b):
    if a < b:
        return -1
    elif a > b:
        return 1
    else:
        return 0
```

## The Ripple Effect

From comparison springs:
- **Sorting**: Arranging by comparison
- **Searching**: Finding through comparison  
- **Optimization**: Choosing the "better"
- **Logic**: True or false, 1 or 0

## The Deep Structure

Comparison requires:
1. **A common measure** - What makes things comparable?
2. **A decision rule** - How do we judge?
3. **Transitivity** - If A > B and B > C, then A > C

## The Philosophical Edge

```python
# When comparison breaks down
class Incomparable:
    def __lt__(self, other):
        raise TypeError("Cannot compare apples to enlightenment")

# When comparison transcends
def quantum_compare(a, b):
    return "both greater and lesser until observed"
```

## See Also
- [[pattern]] - Comparison finds patterns
- [[sorting]] - Comparison creates order
- [[equality]] - The special case of sameness

---
*"To compare is to create hierarchy from chaos."*