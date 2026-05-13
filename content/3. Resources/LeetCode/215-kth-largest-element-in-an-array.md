# 215. Kth Largest Element in an Array

**Date:** 2026-02-02
**Difficulty:** Medium
**Topics:** Heap, Quickselect
**Link:** https://leetcode.com/problems/kth-largest-element-in-an-array/

## Final Solution

```python
def findKthLargest(self, nums: List[int], k: int) -> int:
    min_heap = []
    for num in nums:
        heapq.heappush(min_heap, num)
        if len(min_heap) > k:
            heapq.heappop(min_heap)
    return min_heap[0]
```

## Alternative Solution (Original)

```python
def findKthLargest(self, nums: List[int], k: int) -> int:
    min_heap = []
    for num in nums:
        if len(min_heap) < k:
            heapq.heappush(min_heap, num)
        else:
            if min_heap[0] < num:
                heapq.heappop(min_heap)
                heapq.heappush(min_heap, num)
    return min_heap[0]
```

The original has an optimization: only push/pop when the new element is larger than the current min. Avoids unnecessary operations.

## Initial Issues

- Used `heapify(min_heap)` on empty list - unnecessary (and should be `heapq.heapify()`)

## Key Learnings

### Why Min Heap of Size K Works

1. Keep a min heap that never exceeds size k
2. When it overflows, pop the minimum (smallest element gets kicked out)
3. After processing all elements, heap contains the **k largest elements**
4. Root of min heap = smallest of k largest = **kth largest**

### Time Complexity: O(n log k)

- **n**: Process each element once
- **log k**: Each heap operation on a heap of size k

Better than O(n log n) when k << n.

## Alternative: Quickselect

| Approach | Time (avg) | Time (worst) | Space |
|----------|------------|--------------|-------|
| Min heap of size k | O(n log k) | O(n log k) | O(k) |
| Quickselect | O(n) | O(n²) | O(1) |

**Quickselect idea:**
1. Partition array around a pivot
2. If pivot lands at kth largest position, done
3. Otherwise, recurse on only ONE side (not both)

Faster average case but risky worst case. Heap is safer.

## Nuances to Remember

- Use **min heap** for kth largest (kicks out smallest, keeps k largest)
- Use **max heap** for kth smallest (kicks out largest, keeps k smallest)
- Root of min heap of k largest = kth largest
- `heapq` in Python is a min heap by default

## Q&A Highlights

**Q:** Why min heap and not max heap?
**A:** Min heap kicks out the smallest when it overflows. After seeing all elements, the k survivors are the k largest. The root (minimum of k largest) is the kth largest.

**Q:** Why O(n log k)?
**A:** We iterate n elements, and each heap push/pop is O(log k) because the heap has at most k elements.
