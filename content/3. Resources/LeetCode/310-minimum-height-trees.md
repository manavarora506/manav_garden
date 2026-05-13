# 310. Minimum Height Trees

**Date:** 2026-01-31
**Difficulty:** Medium
**Topics:** BFS, Graph, Topological Sort, Tree
**Link:** https://leetcode.com/problems/minimum-height-trees/

**Status:** ⚠️ REVISIT - Watch tutorial video

## Final Solution

```python
def findMinHeightTrees(self, n: int, edges: List[List[int]]) -> List[int]:
    if n == 1:
        return [0]
    
    adj_list = {i: [] for i in range(n)}
    degree = {i: 0 for i in range(n)}
    
    for curr, nxt in edges:
        adj_list[curr].append(nxt)
        adj_list[nxt].append(curr)
        degree[curr] += 1
        degree[nxt] += 1
    
    queue = deque()
    for node in degree:
        if degree[node] == 1:
            queue.append(node)
    
    remaining = n
    while remaining > 2:
        size = len(queue)
        remaining -= size
        for i in range(size):
            node = queue.popleft()
            for neighbor in adj_list[node]:
                degree[neighbor] -= 1
                if degree[neighbor] == 1:
                    queue.append(neighbor)
    
    return list(queue)
```

## Key Concept

MHT roots are the **center(s)** of the tree. Peel leaves layer by layer until 1-2 nodes remain.

A tree can have at most **2 MHT roots** (middle nodes of the longest path/diameter).

## Bugs I Hit

1. **Initial intuition wrong:** Thought "most edges" = center. Wrong - it's about being equidistant from all leaves.

2. **Used `len(queue) > 2` instead of `remaining > 2`:**
   - `len(queue)` = current leaves only
   - `remaining` = total nodes in tree
   - Line graph `0-1-2-3-4` has queue=[0,4] (size 2) but 5 remaining nodes

3. **Didn't process layer-by-layer:** Must remove ALL current leaves before checking stopping condition, otherwise stop mid-layer.

4. **Forgot edge case:** `n == 1` → return `[0]` (single node has no edges, degree 0)

## Pattern

"Peel the onion" - reverse of Kahn's algorithm:
- Kahn's: start with in-degree 0, peel inward
- MHT: start with degree 1 (leaves), peel inward to find center

## To Review

- [ ] Watch tutorial video to solidify understanding
- [ ] Re-implement from scratch without looking at solution
- [ ] Understand why `remaining` matters vs `len(queue)`
