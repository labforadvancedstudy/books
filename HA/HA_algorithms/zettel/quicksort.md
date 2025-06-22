# quicksort
*L5 • The Elegant Gamble*

Quicksort is algorithms as jazz - improvisation built on solid theory. It's a dance of pivots and partitions, averaging O(n log n) with style.

## The Core Idea

```python
def quicksort(arr):
    if len(arr) <= 1:
        return arr
    
    pivot = arr[len(arr) // 2]  # The chosen one
    left = [x for x in arr if x < pivot]
    middle = [x for x in arr if x == pivot]
    right = [x for x in arr if x > pivot]
    
    return quicksort(left) + middle + quicksort(right)
```

## The In-Place Ballet

```python
def quicksort_inplace(arr, low=0, high=None):
    if high is None:
        high = len(arr) - 1
    
    if low < high:
        # Partition: dance around the pivot
        pivot_index = partition(arr, low, high)
        
        # Recursive elegance
        quicksort_inplace(arr, low, pivot_index - 1)
        quicksort_inplace(arr, pivot_index + 1, high)

def partition(arr, low, high):
    pivot = arr[high]
    i = low - 1  # The boundary of the smaller realm
    
    for j in range(low, high):
        if arr[j] <= pivot:
            i += 1
            arr[i], arr[j] = arr[j], arr[i]  # The swap
    
    arr[i + 1], arr[high] = arr[high], arr[i + 1]
    return i + 1
```

## The Beautiful Complexity

- **Best case**: O(n log n) - Perfect pivots
- **Average case**: O(n log n) - Probability smiles
- **Worst case**: O(n²) - Already sorted! 
- **Space**: O(log n) - Just the call stack

## The Philosophy

Quicksort embodies:
- **Divide and conquer** - Break problems down
- **Randomization** - Embrace uncertainty
- **Cache efficiency** - Spatial locality
- **Practical beats perfect** - Usually faster than "better" algorithms

## The Variations

```python
# Three-way partition for duplicates
def quicksort_3way(arr, low=0, high=None):
    # Handles many duplicates efficiently
    # Dutch National Flag problem
    pass

# Randomized pivot selection
def choose_pivot(arr, low, high):
    return random.randint(low, high)  # Defeat adversaries

# Hybrid approach
def introsort(arr):
    # Start with quicksort
    # Switch to heapsort if recursion too deep
    # Use insertion sort for small subarrays
    pass
```

## The Legacy

Quicksort teaches us:
- **Average case matters** more than worst case
- **Simplicity scales** better than complexity
- **Good enough** is often best
- **Know your data** - patterns matter

## See Also
- [[partition]] - The heart of the algorithm
- [[merge_sort]] - The stable alternative
- [[introsort]] - Quicksort evolved
- [[randomized_algorithms]] - Embracing chance

---
*"Quicksort is the algorithm that knew it was good enough to be great."*