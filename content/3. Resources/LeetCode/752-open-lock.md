# 752. Open the Lock

**Date:** 2026-01-26
**Difficulty:** Medium
**Topics:** BFS, Hash Set, Graph Traversal
**Link:** https://leetcode.com/problems/open-lock/

## Final Solution

```python
class Solution:
    def openLock(self, deadends: List[str], target: str) -> int:
        deadends_set = set(deadends)
        if "0000" in deadends_set:
            return -1
        queue = deque()
        directions = (1, -1)
        queue.append("0000")
        steps = 0
        visited = set()
        visited.add("0000")
        while queue:
            size = len(queue)
            for i in range(size):
                currComb = queue.popleft()
                if currComb == target:
                    return steps
                for j in range(4):
                    for direction in directions:
                        newComb = currComb[0 : j] + str((int(currComb[j]) + direction) % 10) + currComb[j + 1:]
                        if newComb not in deadends_set and newComb not in visited:
                            visited.add(newComb)
                            queue.append(newComb)
            steps += 1
        return -1
```

## Initial Issues

1. **Popping outside the level loop** - Was calling `queue.popleft()` before the `for i in range(size)` loop, so only one node was processed per level instead of all nodes at that level.

2. **Wrong index variable** - Used `i` (the level iteration counter) as the string index instead of `j` (the digit position 0-3).

3. **Checking wrong variable for deadends** - Created `deadends_set` but then checked against `deadends` (the original list) in the condition, making lookups O(n) instead of O(1).

## Key Learnings

### Why track `size` in BFS?
- To know when a "level" ends and the next begins
- Without it, you can't tell when to increment `steps` since the queue keeps growing as you process nodes
- Snapshot the queue size at the start of each level, process exactly that many nodes, then increment

### Alternative: Tuple approach
Instead of tracking size, store `(combination, step_count)` tuples in the queue. Each node carries its own distance. Tradeoff is slightly more memory per queue entry.

### Mark visited when adding vs when popping
- **When adding (correct):** Prevents duplicates in the queue. If A and B both have C as a neighbor, C gets added once.
- **When popping (problematic):** C gets added twice (once from A, once from B) before either is marked visited. Still correct but slower/wasteful.

## Nuances to Remember

- **Python modulo with negatives:** `(-1) % 10 = 9` in Python. The mod operator always returns non-negative when divisor is positive. This handles the 0→9 wraparound correctly.

- **Set initialization shortcut:** `deadends_set = set(deadends)` instead of iterating and adding one by one.

## Q&A Highlights

**Q: What's `(0 + (-1)) % 10`?**
A: It's 9, not 11. Python's modulo handles negative numbers by returning a non-negative result when the divisor is positive.

**Q: Is marking visited when adding vs popping the same?**
A: No. Marking when popping allows duplicates in the queue. If two nodes share a neighbor, that neighbor gets added twice before either pop marks it visited.
