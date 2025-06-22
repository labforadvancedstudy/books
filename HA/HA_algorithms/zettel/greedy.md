# Greedy (L5)

## The Philosophy of Local Excellence

A greedy algorithm makes the locally optimal choice at each stage, hoping to find a global optimum. It's the algorithmic embodiment of "take the best option you can see right now" - a strategy that's surprisingly effective in many situations.

## The Greedy Choice Property

At each step, a greedy algorithm:
1. Examines the current state
2. Makes the best immediate choice
3. Never reconsiders past decisions
4. Moves forward irreversibly

## When Greedy Works

Greedy algorithms work when:
- Local optimal choices lead to global optimal solutions
- The problem has "optimal substructure"
- Past choices don't constrain future options negatively

## Classic Examples

```python
# Making change with coins (works for standard US coins)
def make_change(amount, coins=[25, 10, 5, 1]):
    result = []
    for coin in coins:
        count = amount // coin
        if count > 0:
            result.extend([coin] * count)
            amount %= coin
    return result

# Huffman coding - greedy frequency-based encoding
def huffman_encoding(frequencies):
    heap = [[freq, [char, ""]] for char, freq in frequencies.items()]
    heapify(heap)
    
    while len(heap) > 1:
        lo = heappop(heap)
        hi = heappop(heap)
        for pair in lo[1:]:
            pair[1] = '0' + pair[1]
        for pair in hi[1:]:
            pair[1] = '1' + pair[1]
        heappush(heap, [lo[0] + hi[0]] + lo[1:] + hi[1:])
    
    return sorted(heappop(heap)[1:], key=lambda p: (len(p[-1]), p))
```

## Real-World Greedy Thinking

- **Navigation**: Turn toward your destination at each intersection
- **Investing**: Buy the stock with highest current return
- **Eating**: Choose the most appetizing food first
- **Task Scheduling**: Always do the shortest task next

## When Greedy Fails

The traveling salesman who greedily visits the nearest city might traverse the globe inefficiently. Greedy doesn't always work because:
- Local optimum ≠ Global optimum
- Early choices can trap you
- The best path might require temporary sacrifices

## The Beauty of Greedy

When it works, greedy is elegant:
- Simple to understand
- Easy to implement
- Often very efficient
- No need to explore all possibilities

## Greedy vs Other Approaches

- **Dynamic Programming**: Considers all options, greedy doesn't
- **Brute Force**: Tries everything, greedy tries one path
- **Divide & Conquer**: Breaks down problems, greedy builds up solutions

## Life Lessons from Greedy

The greedy approach mirrors how we often navigate life:
- Making the best decision with current information
- Moving forward without dwelling on past choices
- Sometimes missing the bigger picture
- Often good enough, rarely perfect

## Famous Greedy Algorithms

1. **Dijkstra's Algorithm**: Shortest path in graphs
2. **Kruskal's/Prim's**: Minimum spanning trees
3. **Huffman Coding**: Optimal prefix codes
4. **Activity Selection**: Maximum non-overlapping activities

## Connection to Other Concepts

- **Optimization** (L4): Greedy is one optimization strategy
- **Making Choices** (L0): The everyday version of greedy
- **Dynamic Programming** (L5): The alternative to greedy
- **Graph** (L3): Where many greedy algorithms operate