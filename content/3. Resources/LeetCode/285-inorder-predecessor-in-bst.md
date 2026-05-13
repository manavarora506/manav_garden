# 285. Inorder Predecessor in BST

**Date:** 2026-02-17
**Difficulty:** Medium
**Topics:** BST, Binary Search
**Link:** https://leetcode.com/problems/inorder-predecessor-in-bst/

## Final Solution

```python
def inorderPredecessor(self, root: TreeNode, p: TreeNode) -> Optional[TreeNode]:
    predecessor = None
    while root:
        if root.val < p.val:
            predecessor = root
            root = root.right
        else:
            root = root.left
    return predecessor
```

## Key Learnings

- Exact mirror of inorder successor
- When going right (root.val < p.val), that node is a potential predecessor
- When root.val >= p.val, go left to find something smaller
- Got it right on the first attempt by applying the successor pattern

## Nuances to Remember

- O(H) time, O(1) space — same as successor
- Successor: update when going left; Predecessor: update when going right
