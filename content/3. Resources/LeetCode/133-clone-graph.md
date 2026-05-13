# 133. Clone Graph

**Date:** 2026-02-02
**Difficulty:** Medium
**Topics:** DFS, Graph Traversal, Hash Map
**Link:** https://leetcode.com/problems/clone-graph/

## Final Solution

```python
def cloneGraph(self, node: Optional['Node']) -> Optional['Node']:
    node_to_copy = {}
    
    def dfs(node: Node):
        if not node:
            return None
        if node in node_to_copy:
            return node_to_copy[node]
        cloned_node = Node(node.val)
        node_to_copy[node] = cloned_node
        for child in node.neighbors:
            cloned_node.neighbors.append(dfs(child))
        return cloned_node

    return dfs(node)
```

## Initial Issues

- **Class-level dict**: Initially defined `node_to_copy = {}` as class variable, which persists between test cases
- **Missing None check**: Forgot base case for empty graph
- **Missing return statement**: Forgot to return `cloned_node` at end of `dfs` function
- **Wrong recursive call**: Used `cloneGraph(child)` instead of `self.cloneGraph(child)` or `dfs(child)`

## Key Learnings

### The Chicken-and-Egg Problem
When cloning node 1, its neighbors should point to cloned node 2 and cloned node 4. But those clones don't exist yet when you start cloning node 1.

**Solution:** Use a dict mapping `original node → cloned node`. Build clones incrementally.

### Why the Dict is Necessary

**1. Prevents infinite recursion from cycles:**
```
Without dict: clone(1) → clone(2) → clone(1) → clone(2) → ...
With dict: clone(1) → clone(2) → node 1 in dict → return existing clone
```

**2. Ensures single clone per node:**
```
1 --- 2
|     |
4 --- 3
```
Nodes 1 and 3 both neighbor node 2. Without dict, you'd create two different clones of 2. With dict, both reference the same clone2.

### Critical: Add to Dict BEFORE Recursing

```python
cloned_node = Node(node.val)
node_to_copy[node] = cloned_node  # ← BEFORE recursing!
for child in node.neighbors:
    cloned_node.neighbors.append(dfs(child))
```

If you add after recursing, cycles cause infinite recursion because the node isn't in the dict when you encounter it again.

## Nuances to Remember

| Step | Why |
|------|-----|
| Check `if not node` first | Handle empty graph |
| Check `if node in dict` | Return existing clone, prevent re-cloning |
| Add to dict before recursing | Prevent infinite loops on cycles |
| Return the cloned node | Caller needs it to set as neighbor |

## Pattern: Graph Cloning

This pattern applies whenever you need to deep copy a graph structure:
1. Use dict: `original → copy`
2. Add to dict before recursing (handles cycles)
3. Recursively clone children, looking up existing clones from dict
