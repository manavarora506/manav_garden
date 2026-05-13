# 429. N-ary Tree Level Order Traversal

**Date:** 2026-01-31
**Difficulty:** Medium
**Topics:** BFS, Tree, Level Order Traversal
**Link:** https://leetcode.com/problems/n-ary-tree-level-order-traversal/

## Final Solution

```python
class Solution:
    def levelOrder(self, root: 'Node') -> List[List[int]]:
        result = []
        if not root:
            return result
        queue = deque()
        queue.append(root)
        while queue:
            size = len(queue)
            curr_level = []
            for i in range(size):
                node = queue.popleft()
                curr_level.append(node.val)
                for child in node.children:
                    queue.append(child)
            result.append(curr_level)
        return result
```

## Initial Issues

- None - solved cleanly on first attempt

## Key Learnings

- **N-ary vs Binary tree traversal:** Only difference is how you add children
  - Binary: `if node.left: queue.append(node.left)` + same for right
  - N-ary: `for child in node.children: queue.append(child)`
- Same BFS level-order pattern applies regardless of tree structure

## Nuances to Remember

- `node.children` is a list (possibly empty), so iterating over it handles the "no children" case naturally
- No need to check `if node.children` before the loop - empty list just means zero iterations

## Pattern Summary

Fourth BFS problem in session - same core structure, different collection per level:

| Problem | What you collect/do per level |
|---------|-------------------------------|
| 116 - Next Pointers | Connect nodes via `.next` |
| 1091 - Shortest Path | Count steps (distance) |
| 429 - N-ary Traversal | Node values into lists |
