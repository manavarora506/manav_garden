# 797. All Paths From Source to Target

**Date:** 2026-01-27
**Difficulty:** Medium
**Topics:** BFS, DFS, Backtracking, Graph Traversal
**Link:** https://leetcode.com/problems/all-paths-from-source-to-target/

## Final Solution

```python
class Solution:
    def allPathsSourceTarget(self, graph: List[List[int]]) -> List[List[int]]:
        nodes = len(graph)
        dest = nodes - 1
        queue = deque()
        queue.append((0, [0]))
        valid_paths = []
        while queue:
            curr, currPath = queue.popleft()
            if curr == dest:
                valid_paths.append(currPath)
            for edge in graph[curr]:
                edgePath = currPath.copy()
                edgePath.append(edge)
                queue.append((edge, edgePath))
        return valid_paths
```

## Initial Issues

- **Path reference bug:** Was using the same list reference for all paths instead of copying. All paths ended up sharing/mutating the same list.
- **Unnecessary adjacency list construction:** Built a separate `adj_list` dictionary when the input `graph` is already in adjacency list format (`graph[i]` gives neighbors of node `i`).
- **Unused visited set:** Initially declared `visited = {0}` but realized it wasn't needed.

## Key Learnings

- **DAGs don't need visited tracking for all-paths problems:** Since there are no cycles, you can't get stuck in infinite loops. The same node CAN be visited on different paths (and should be).
- **Input format awareness:** Always check what the input already provides before building additional data structures.

## Nuances to Remember

- When exploring multiple branches, each branch needs its own copy of the path state
- `.append()` returns `None` and mutates in place - can't chain it in expressions
- In DAGs: no cycles means no infinite loops, but same node can appear in multiple valid paths

## Q&A Highlights

- **Q: Why don't we need a visited set?**  
  A: DAG = no cycles. You can never revisit a node on the same path. But you CAN (and should) visit the same node on different paths.

- **Q: Is BFS easier for finding all paths?**  
  A: DFS with backtracking is often preferred because it copies the path less often (only when a valid path is found, not at every branch). BFS copies at every branching point. Backtracking pattern: choose → explore → unchoose.

## Next Steps

- Practice DFS backtracking version of this problem
- Explore backtracking pattern in other problems (permutations, combinations, N-Queens)
# 797. All Paths From Source to Target

**Date:** 2026-02-02
**Difficulty:** Medium
**Topics:** DFS, Backtracking, Graph Traversal
**Link:** https://leetcode.com/problems/all-paths-from-source-to-target/

## Final Solution

```python
def allPathsSourceTarget(self, graph: List[List[int]]) -> List[List[int]]:
    valid_paths = []
    n = len(graph)
    
    def dfs(i, currPath):
        if i == n - 1:
            valid_paths.append(currPath + [n - 1])
            return
        currPath.append(i)
        for edge in graph[i]:
            dfs(edge, currPath)
        currPath.pop()
    
    dfs(0, [])
    return valid_paths
```

## Initial Issues

- **Starting DFS from all nodes**: Originally looped through all nodes instead of starting from node 0 only
- **Missing parameter in recursive call**: Had `dfs(edge)` instead of `dfs(edge, currPath)`
- **Missing backtrack on visited set**: Had `visited.add(i)` but forgot `visited.remove(i)`

## Key Learnings

- **DAG doesn't need visited set**: Since directed acyclic graphs have no cycles, you can't revisit a node through a loop. The `currPath` tracking is for building paths, not preventing cycles.
- **General graphs need visited set**: If cycles are possible, `visited` prevents infinite loops
- **Backtracking enables multiple paths**: `currPath.pop()` after exploring lets other branches use that node

## Nuances to Remember

| Graph Type | Need `visited` set? | Need `currPath` backtracking? |
|------------|---------------------|-------------------------------|
| DAG | No | Yes |
| General graph (may have cycles) | Yes | Yes |

- `currPath + [n - 1]` creates a new list, so no need for `.copy()`
- Backtrack **after** the for loop, not inside it

## Q&A Highlights

**Q:** Why don't we need a visited set for this problem?
**A:** It's a DAG - no cycles exist. The only way to reach a node twice is via different paths, which is exactly what we want to find.

**Q:** When would we need the visited set?
**A:** In a general graph where cycles are possible. The visited set prevents following a cycle back to a node already in the current path.
