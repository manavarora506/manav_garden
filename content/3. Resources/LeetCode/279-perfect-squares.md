# 279. Perfect Squares

**Date:** 2026-01-26
**Difficulty:** Medium
**Topics:** BFS, Graph Modeling, Dynamic Programming
**Link:** https://leetcode.com/problems/perfect-squares/

## Final Solution

```python
def numSquares(self, n: int) -> int:
    valid_squares = []
    i = 1
    while i * i <= n:
        valid_squares.append(i*i)
        i += 1
    queue = deque()
    queue.append(n)
    steps = 0
    visited = {n}
    while queue:
        size = len(queue)
        for i in range(size):
            curr = queue.popleft()
            for square in valid_squares:
                newNum = curr - square
                if newNum < 0:
                    break  # squares are ascending, rest will also be negative
                if newNum == 0:
                    return steps + 1
                if newNum not in visited:
                    visited.add(newNum)
                    queue.append(newNum)
        steps += 1
    return steps
```

## Initial Issues

1. **Loop condition off-by-one** - Used `while i * i < n` instead of `while i * i <= n`, missing perfect squares equal to n.

2. **Wrong set initialization** - Used `set(n)` which throws an error (expects iterable). Fixed to `{n}` or `set([n])`.

3. **Missing visited check** - Added to visited but didn't check before adding, causing duplicates in queue.

4. **Missing negative check** - Didn't handle cases where `curr - square < 0`.

## Key Learnings

### The Core Insight: Graph Modeling
The challenge was visualizing this as a graph problem:
- **Nodes** = numbers (n, n-1, ..., 0)
- **Edges** = connect numbers that differ by a perfect square
- **Target** = 0
- **Goal** = shortest path from n to 0

Once framed this way, BFS writes itself.

### Pattern Recognition
When a problem asks for:
- "Minimum number of operations to transform X into Y"
- "Fewest moves to reach a target"
- "Least steps to achieve some state"

Think: **shortest path in disguise**. Identify nodes, edges, and target.

## Nuances to Remember

- **`continue` vs `break`**: `continue` skips to next iteration; `break` exits the loop entirely. Since `valid_squares` is ascending, once one square is too big, all remaining are too - use `break`.

- **Precompute valid squares** once before BFS, not inside the loop.

- **DP alternative exists**: `dp[i] = min(dp[i - j*j] + 1) for all j*j <= i`

- **Math fact**: Lagrange's theorem - every positive integer is sum of at most 4 squares. Answer is always 1-4.

## Q&A Highlights

**Q: How is this a graph problem?**
A: Reframe as "get from n down to 0, each perfect square is a step." Target = 0 was the unlock.

**Q: What's the difference between `continue` and `break`?**
A: `continue` skips rest of current iteration, moves to next. `break` exits the loop entirely.
