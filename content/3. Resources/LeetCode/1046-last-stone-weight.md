# 1046. Last Stone Weight

**Date:** 2026-02-02
**Difficulty:** Easy
**Topics:** Heap, Simulation
**Link:** https://leetcode.com/problems/last-stone-weight/

## Final Solution

```python
def lastStoneWeight(self, stones: List[int]) -> int:
    max_heap = [-stone for stone in stones]
    heapq.heapify(max_heap)
    while len(max_heap) > 1:
        y = -(heapq.heappop(max_heap))
        x = -(heapq.heappop(max_heap))
        if x != y:
            newStone = y - x
            heapq.heappush(max_heap, -newStone)
    if len(max_heap) > 0:
        return -(max_heap[0])
    else:
        return 0
```

## Initial Issues

1. **`heapify` returns None**: Wrote `max_heap = heapq.heapify([...])` which sets max_heap to None. Fix: create list first, then call heapify in place.

2. **Wrong while condition**: Used `len(max_heap) >= 1` but need at least 2 stones to smash. Fix: `len(max_heap) > 1`

3. **Missing heap argument**: Wrote `heapq.heappop()` without passing the heap. Fix: `heapq.heappop(max_heap)`

4. **Backwards return logic**: Had `if len == 0: return max_heap[0]` which errors on empty list. Fix: check `> 0` first.

## Key Learnings

### Max Heap in Python

Python's `heapq` is a min heap. For max heap, negate values:
- Push: `heappush(heap, -value)`
- Pop: `-(heappop(heap))`

### Correct Heap Initialization

```python
# WRONG - heapify returns None
max_heap = heapq.heapify([-s for s in stones])

# RIGHT - heapify modifies in place
max_heap = [-s for s in stones]
heapq.heapify(max_heap)
```

Option 1 (heapify) is O(n). Pushing n elements individually is O(n log n).

## Nuances to Remember

- `heapify()` modifies the list **in place** and returns `None`
- `heappush()` also returns `None`
- Need at least 2 elements to do a "smash" operation
- When returning from max heap, negate the value back: `-(max_heap[0])`

## Q&A Highlights

**Q:** Why use a max heap?
**A:** Need to repeatedly get the two heaviest stones. Max heap gives O(log n) access to the maximum.

**Q:** What's the time complexity?
**A:** O(n log n) - potentially n smash operations, each with O(log n) push/pop.
