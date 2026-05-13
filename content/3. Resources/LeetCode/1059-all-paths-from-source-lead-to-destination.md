# 1059. All Paths from Source Lead to Destination

**Date:** 2026-02-02
**Difficulty:** Medium
**Topics:** DFS, Graph Traversal, Cycle Detection, Topological Sort
**Link:** https://leetcode.com/problems/all-paths-from-source-lead-to-destination/

## Final Solution (DFS with Memoization)

```python
def leadsToDestination(self, n: int, edges: List[List[int]], source: int, destination: int) -> bool:
    adj_list = defaultdict(list)
    visited = set()    # in current path (gray)
    verified = set()   # fully processed (black)
    
    for start, end in edges:
        adj_list[start].append(end)
    
    def dfs(node):
        if node in verified:
            return True   # already confirmed leads to destination
        if node in visited:
            return False  # cycle detected
        if not adj_list[node]:  # terminal node
            return node == destination
        
        visited.add(node)
        for neighbor in adj_list[node]:
            if not dfs(neighbor):
                visited.remove(node)
                return False
        visited.remove(node)
        verified.add(node)
        return True

    return dfs(source)
```

## Initial Issues

- **Early return at destination without checking terminal**: Initially returned True when reaching destination without checking if it had outgoing edges
- **TLE without memoization**: Used only `visited` set, causing redundant re-exploration of already-verified nodes
- **Fix**: Added `verified` set to cache nodes confirmed to lead to destination

## Key Learnings

- **Two sets for directed graph traversal**:
  - `visited` (gray): currently in path, for cycle detection, gets backtracked
  - `verified` (black): fully processed, permanent, prevents re-exploration

- **Terminal node check**: `if not adj_list[node]: return node == destination`
  - Combines dead-end and destination check in one line

- **Destination must be terminal**: If destination has outgoing edges, paths continue past it

## Nuances to Remember

| Set | Purpose | Backtracked? |
|-----|---------|--------------|
| `visited` / `in_path` | Cycle detection | Yes (remove after exploring) |
| `verified` | Memoization | No (permanent) |

- Check `verified` BEFORE `visited` to short-circuit early
- Add to `verified` only after ALL neighbors return True

## Alternative: Topological Sort Approach

Can also solve with reverse Kahn's algorithm:
1. Find all nodes reachable from source
2. Destination must be the only terminal node (out-degree 0)
3. Process backwards from terminal nodes, decrementing predecessor out-degrees
4. If all reachable nodes get processed → no cycle → return True

The DFS with memoization is simpler, but topo sort shows the DAG connection.

## Q&A Highlights

**Q:** Why did the initial solution get TLE?
**A:** Without `verified`, we re-explore nodes multiple times. In graph `0→1→3, 0→2→1→3`, node 1 gets fully explored twice without memoization.

**Q:** Can we solve with topological sort?
**A:** Yes - use reverse Kahn's (out-degree instead of in-degree). Start from terminal nodes, work backwards. If all reachable nodes can be processed, no cycle exists and all paths lead to destination.

**Q:** What's the difference between `visited` and `verified`?
**A:** `visited` = "I'm currently here, coming back means cycle" (temporary). `verified` = "I already checked all paths from here, they all lead to destination" (permanent).


## Why DFS Over BFS for Directed Cycle Detection?

### The Core Problem

**Undirected graph:** Any visited node = cycle
- BFS works fine - if you see a visited node, you found a cycle

**Directed graph:** Visited node ≠ cycle
- Two paths can converge at the same node without forming a cycle
- You need to know if a node is **in your current path** (ancestor), not just "ever visited"

### Example: Why BFS Fails for Directed Graphs

```
A → B
↓   ↓
C → D
```

BFS from A:
- Level 0: Visit A
- Level 1: Visit B, C
- Level 2: Visit D from B, mark visited
- Level 2: C tries to visit D - already visited!

Is this a cycle? **No.** Two paths (A→B→D and A→C→D) just converge at D.

BFS only knows "visited or not" - it can't distinguish between:
- Visiting an ancestor (cycle)
- Visiting a node reached via a different path (not a cycle)

### Why DFS Works

DFS naturally tracks the **current path** via the call stack:

```
dfs(A)           ← stack: [A]
  dfs(B)         ← stack: [A, B]
    dfs(D)       ← stack: [A, B, D]
    return
  dfs(C)         ← stack: [A, C]
    dfs(D)       ← D visited but NOT in [A, C], so NOT a cycle
```

The `visited` set with backtracking mimics this stack:
- Add node when entering (gray)
- Remove node when leaving (black)
- If you see a gray node → it's in current path → cycle

### When to Use Each

| Graph Type | BFS | DFS |
|------------|-----|-----|
| Undirected cycle detection | Works | Works |
| Directed cycle detection | Inefficient | Preferred |
| Shortest path (unweighted) | Preferred | Works but not optimal |
| Exhaustive search / all paths | Possible | Preferred |

### Memory Consideration

BFS might use less memory for "deep and narrow" graphs:
```
A → B → C → D → E → F → G → H
```
- DFS stack: up to 8 frames
- BFS queue: 1-2 nodes at a time

But this is rare and doesn't solve the directed cycle detection problem.

### Summary

DFS wins for directed cycle detection because **the recursion stack IS the current path**. BFS would need to store the full path for each node in the queue, which is expensive and unnatural.
