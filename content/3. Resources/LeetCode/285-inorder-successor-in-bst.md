# 285. Inorder Successor in BST

**Date:** 2026-02-17
**Difficulty:** Medium
**Topics:** BST, Binary Search
**Link:** https://leetcode.com/problems/inorder-successor-in-bst/

## Final Solution

```python
def inorderSuccessor(self, root: TreeNode, p: TreeNode) -> Optional[TreeNode]:
    successor = None
    while root:
        if root.val > p.val:
            successor = root
            root = root.left
        else:
            root = root.right
    return successor
```

## Initial Issues

- First definition was incomplete: "right child of a node" — missed the leftmost-in-right-subtree case and the no-right-subtree case
- Initial recursive solution had unreachable `return successor` after if/else branches
- Redundant `if not root` check inside `while root` loop

## Key Learnings

- Inorder successor has two cases:
  1. **Has right subtree**: go right, then left as far as possible
  2. **No right subtree**: walk up to find ancestor where node is in its left subtree
- Without parent pointers, search from root downward — every time you go left (node.val > p.val), that node is a potential successor
- When node.val == p.val, go right since successor must be greater

## Nuances to Remember

- O(H) time because you follow a single root-to-leaf path — never backtrack or visit siblings
- O(H) is O(log n) for balanced BST, O(n) for skewed
- O(1) space with iterative approach
- The iterative version is cleaner than recursive for this problem — no need for nonlocal or helper functions

## Q&A Highlights

- **Q: Why O(H)?** Each loop iteration moves one level down (left or right), never revisiting. Max iterations = height of tree.
- **Q: What if node.val == p.val?** Go right — successor must be strictly greater.
