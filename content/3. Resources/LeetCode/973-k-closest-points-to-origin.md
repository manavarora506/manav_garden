# 973. K Closest Points to Origin

**Date:** 2026-02-02
**Difficulty:** Medium
**Topics:** Heap, Math
**Link:** https://leetcode.com/problems/k-closest-points-to-origin/

## Final Solution

```python
def kClosest(self, points: List[List[int]], k: int) -> List[List[int]]:
    def _calcDist(point):
        x, y = point[0], point[1]
        return x * x + y * y  # No sqrt needed
    
    max_heap = []
    res = []
    for point in points:
        ed = _calcDist(point)
        heapq.heappush(max_heap, (-ed, point))
        if len(max_heap) > k:
            heapq.heappop(max_heap)
    while max_heap:
        ed, point = heapq.heappop(max_heap)
        res.append(point)
    return res
```

## Initial Issues

1. **Typo in function name**: `_caclEuclidDist` vs `_calcEuclidDist`

2. **Wrong tuple order**: Had `(point, ed)` but heap compares first element. Need `(ed, point)` to compare by distance.

3. **Not using max heap**: Forgot to negate distance. Need `-ed` for max heap behavior.

4. **Overcomplicated pop logic**: Had `if max_heap[0] > ed` comparing tuple to float. Just pop when size > k.

5. **Wrong heappop syntax**: `max_heap.heappop(max_heap)` should be `heapq.heappop(max_heap)`

## Key Learnings

### K Closest vs K Largest - Which Heap?

| Goal | Heap type | Kick out | Keep |
|------|-----------|----------|------|
| K largest | Min heap | Smallest | K largest |
| K closest (smallest) | Max heap | Largest | K smallest |

For k **closest** (smallest distances), use **max heap** - kick out the largest distance to keep k smallest.

### No Sqrt Needed for Comparison

`sqrt` is monotonic: if `a < b`, then `sqrt(a) < sqrt(b)`.

So `x² + y²` is enough for comparing distances. Skip the `sqrt` for efficiency.

### Tuple Comparison in Heaps

Python compares tuples element by element. Put the comparison key first:
```python
heapq.heappush(heap, (distance, point))  # Compares by distance
```

## Nuances to Remember

### Safe Tuple Pattern with Index

If two distances are equal, Python compares the next element (point). This can fail for non-comparable types.

**Safe pattern - add unique tiebreaker:**
```python
for i, point in enumerate(points):
    heapq.heappush(heap, (-dist, i, point))
```

Indices are unique, so comparison never reaches `point`.

### Heap Tuple Order Matters

```python
(-ed, point)  # Compares by -ed (what we want)
(point, -ed)  # Compares by point first (wrong!)
```

## Q&A Highlights

**Q:** Why max heap for k closest?
**A:** We want to keep k smallest distances. When heap overflows, max heap kicks out the largest distance, keeping the k closest points.

**Q:** Why no sqrt?
**A:** sqrt is monotonic - relative ordering is preserved. x² + y² is enough for comparison.
