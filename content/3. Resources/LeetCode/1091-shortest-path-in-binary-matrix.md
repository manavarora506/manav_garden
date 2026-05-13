# 1091. Shortest Path in Binary Matrix

**Date:** 2026-01-31
**Difficulty:** Medium
**Topics:** BFS, Shortest Path, Matrix, Graph
**Link:** https://leetcode.com/problems/shortest-path-in-binary-matrix/

## Final Solution

```python
class Solution:
    def shortestPathBinaryMatrix(self, grid: List[List[int]]) -> int:
        n = len(grid)
        if grid[0][0] != 0 or grid[n-1][n-1] != 0:
            return -1
        queue = deque()
        visited = set()
        directions = [(0,1), (0,-1), (1,0), (-1,0), (1,1), (1,-1), (-1,1), (-1,-1)]
        steps = 1
        queue.append((0, 0))
        visited.add((0, 0))
        while queue:
            size = len(queue)
            for i in range(size):
                curr = queue.popleft()
                cx, cy = curr
                if cx == n - 1 and cy == n - 1:
                    return steps
                for direction in directions:
                    nx, ny = cx + direction[0], cy + direction[1]
                    if 0 <= nx < n and 0 <= ny < n and (nx, ny) not in visited and grid[nx][ny] == 0:
                        queue.append((nx, ny))
                        visited.add((nx, ny))
            steps += 1
        return -1
```

## Alternative: O(1) Space (Mutate Grid)

```python
# Instead of visited set, mark cells as blocked:
grid[nx][ny] = 1  # replaces visited.add((nx, ny))
```

## Initial Issues

- None significant - solved cleanly on first attempt

## Key Learnings

- **BFS guarantees shortest path** in unweighted graphs because it explores all nodes at distance k before any at distance k+1
- **8 directions** for diagonal movement: include (1,1), (1,-1), (-1,1), (-1,-1)
- **Mark visited when adding to queue**, not when popping - prevents duplicate entries in queue
- **Path length includes start cell**, so initialize steps = 1

## Nuances to Remember

- Edge case: check if start `(0,0)` or end `(n-1,n-1)` is blocked before BFS
- Can mutate grid (`grid[nx][ny] = 1`) instead of using visited set for O(1) extra space
- Trade-off: mention in interview "I could modify the grid to save space, but that mutates input"
- Tuple unpacking: `cx, cy = curr` is cleaner than `cx, cy = curr[0], curr[1]`

## Q&A Highlights

- **Q: Why BFS instead of DFS?**  
  A: BFS explores level by level, guaranteeing shortest path. DFS would find *a* path but not necessarily the shortest.

- **Q: How to avoid the visited set?**  
  A: Set `grid[nx][ny] = 1` to mark as "blocked" - the grid becomes the visited tracker.

## Pattern Recognition

This is the third BFS problem in the session:
1. **797 - All Paths (DAG):** Track full paths, not just visited
2. **116 - Next Pointers:** Level-order for connecting nodes
3. **1091 - Shortest Path:** Classic BFS distance finding

Core pattern is the same; what varies is what you track (paths vs distance vs connections).
