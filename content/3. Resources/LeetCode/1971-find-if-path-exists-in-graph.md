# 1971. Find if Path Exists in Graph

**Date:** 2026-01-26
**Difficulty:** Easy
**Topics:** BFS, Graph Traversal, Undirected Graph
**Link:** https://leetcode.com/problems/find-if-path-exists-in-graph/

## Final Solution

```python
class Solution:
    def validPath(self, n: int, edges: List[List[int]], source: int, destination: int) -> bool:
        adj_list = {i: [] for i in range(n)}
        for edge in edges:
            src, dest = edge[0], edge[1]
            adj_list[src].append(dest)
            adj_list[dest].append(src)
        visited = {source}
        queue = deque()
        queue.append(source)
        while queue:
            curr = queue.popleft()
            if curr == destination:
                return True
            for edge in adj_list[curr]:
                if edge not in visited:
                    visited.add(edge)
                    queue.append(edge)
        return False
```

## Initial Issues

None - solved correctly on first attempt by applying patterns from previous problems.

## Key Learnings

### Standard BFS for Path Existence

This is the straightforward application of BFS:
1. Build adjacency list (both directions for undirected graph)
2. Start from source, explore neighbors
3. Return True when destination found
4. Return False if queue empties

### Patterns Applied from Previous Problems

- **Mark visited when adding** (from Open Lock) - prevents duplicates in queue
- **Undirected graph means add both directions** - `adj_list[src].append(dest)` AND `adj_list[dest].append(src)`
- **Initialize visited with starting node** - don't forget to include the source

## Nuances to Remember

- For undirected graphs, always add edges in both directions
- This problem could also be solved with Union-Find (check if source and destination are in same connected component)
- DFS would work equally well here - just existence check, not shortest path

## Q&A Highlights

No major struggles on this one - the BFS fundamentals from earlier problems transferred directly.
# 1971. Find if Path Exists in Graph

**Date:** 2026-02-02
**Difficulty:** Easy
**Topics:** DFS, Graph Traversal
**Link:** https://leetcode.com/problems/find-if-path-exists-in-graph/

## Final Solution

```python
def validPath(self, n: int, edges: List[List[int]], source: int, destination: int) -> bool:
    def dfs(source: int, path: set) -> bool:
        if source == destination:
            return True
        path.add(source)
        for edge in adj_list[source]:
            if edge not in path:
                if dfs(edge, path):
                    return True
        return False

    adj_list = {i: [] for i in range(n)}
    for start, end in edges:
        adj_list[start].append(end)
        adj_list[end].append(start)
    path = set()
    return dfs(source, path)
```

## Initial Issues

- **Returning too early in for loop**: Original code had `return dfs(edge, path)` which immediately returned the result of the first neighbor's DFS, even if it was False. This meant other neighbors were never explored.
- **Fix**: Changed to `if dfs(edge, path): return True` so we only return True when found, otherwise continue trying other neighbors.

## Key Learnings

- **DFS existence pattern**: When checking if something exists, use the pattern:
  ```python
  for neighbor in neighbors:
      if dfs(neighbor):
          return True
  return False
  ```
- **"Propagate True up"**: When destination is found, True returns to caller, who also returns True, bubbling up the entire call stack.

## Nuances to Remember

- **Shared visited set**: The visited set is mutable and shared across all recursive calls - no need to pass it explicitly (could use outer scope)
- **No backtracking needed for existence check**: Once a node is explored and doesn't lead to destination, revisiting it from a different path won't change that result
- **When to backtrack**: Only needed when counting all paths or finding all solutions - same node can appear in different paths

| Problem Type | Backtrack? | Why |
|--------------|-----------|-----|
| Path exists? | No | Once explored, result won't change |
| Count all paths | Yes | Same node can appear in different paths |
| Find shortest path | Use BFS | DFS isn't ideal for this |

## Q&A Highlights

**Q:** Why don't we need to backtrack / remove nodes from visited set?
**A:** For existence checks, if node X doesn't lead to destination when explored from node A, it won't lead to destination from node B either. Leaving it visited is actually an optimization.

**Q:** When would we need backtracking?
**A:** When counting all paths or finding all solutions. Example: counting paths in a graph where `0→1→3` and `0→2→3` are both valid - without backtracking you'd only count one.
