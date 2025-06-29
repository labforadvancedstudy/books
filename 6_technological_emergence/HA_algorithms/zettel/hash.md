# hash
*L3 • The Alchemical Transform*

A hash function is computational alchemy - transforming any input into a fixed-size fingerprint. It's how we create order from chaos, enabling O(1) lookup in a universe of possibilities.

## The Magic

```python
# The simplest hash - modulo
def simple_hash(key, table_size):
    return key % table_size

# The reality - complex transforms
def better_hash(key):
    hash_value = 0
    for char in str(key):
        hash_value = (hash_value * 31 + ord(char)) % BIG_PRIME
    return hash_value
```

## The Contract

A good hash function promises:
- **Deterministic**: Same input → same output
- **Uniform**: Spread values evenly
- **Fast**: O(1) computation
- **Avalanche**: Small input change → large output change

## The Applications

```python
# The hash table - O(1) average lookup
class HashTable:
    def __init__(self, size=1000):
        self.buckets = [[] for _ in range(size)]
    
    def put(self, key, value):
        bucket = self.buckets[hash(key) % len(self.buckets)]
        bucket.append((key, value))
    
    def get(self, key):
        bucket = self.buckets[hash(key) % len(self.buckets)]
        for k, v in bucket:
            if k == key:
                return v
        return None
```

## The Deep Magic

Hashing enables:
- **Sets**: Is this element present? O(1)
- **Caches**: Have we computed this before? O(1)
- **Deduplication**: Have we seen this? O(1)
- **Cryptography**: One-way transformations

## The Philosophical Paradox

```python
# The birthday paradox in action
def collision_probability(n_items, table_size):
    # With just 23 people, >50% chance of birthday collision
    # With sqrt(table_size) items, collisions become likely
    return 1 - math.exp(-n_items**2 / (2 * table_size))
```

## The Deeper Truth

Hashing is:
- **Lossy compression**: Infinite → finite
- **Controlled collision**: Accepting imperfection
- **Trading space for time**: Memory → speed
- **Digital fingerprinting**: Identity reduced

## See Also
- [[collision]] - When hashes clash
- [[bloom_filter]] - Probabilistic hashing
- [[consistent_hashing]] - Distributed systems
- [[cryptographic_hash]] - One-way functions

---
*"To hash is to accept that infinity must fit in finite space - and somehow, it does."*