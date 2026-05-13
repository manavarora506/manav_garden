# 701. Insert into a Binary Search Tree

**Date:** 2026-02-17
**Difficulty:** Medium
**Topics:** BST
**Link:** https://leetcode.com/problems/insert-into-a-binary-search-tree/

## Final Solution

### Iterative

```python
def insertIntoBST(self, root: Optional[TreeNode], val: int) -> Optional[TreeNode]:
    if not root:
        return TreeNode(val)
    parent = None
    orig = root
    while root:
        parent = root
        if root.val > val:
            root = root.left
        elif root.val < val:
            root = root.right
    if parent.val > val:
        parent.left = TreeNode(val)
    else:
        parent.right = TreeNode(val)
    return orig
```

### Recursive

```python
def insertIntoBST(self, root: Optional[TreeNode], val: int) -> Optional[TreeNode]:
    if not root:
        return TreeNode(val)
    if root.val > val:
        root.left = self.insertIntoBST(root.left, val)
    else:
        root.right = self.insertIntoBST(root.right, val)
    return root
```

## Key Learnings

- New nodes always get inserted as leaves
- Recursive version is simpler — no parent tracking needed because `root.left = ...` handles attachment when recursion returns the new node
- Iterative needs a `parent` pointer and `orig` to remember the root

## Nuances to Remember

- Iterative: O(H) time, O(1) space
- Recursive: O(H) time, O(H) space due to call stack
- Multiple valid insertions exist; inserting as a leaf is simplest
