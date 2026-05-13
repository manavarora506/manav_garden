# 173. Binary Search Tree Iterator

**Date:** 2026-02-17
**Difficulty:** Medium
**Topics:** BST, Stack, Iterative Inorder Traversal
**Link:** https://leetcode.com/problems/binary-search-tree-iterator/

## Final Solution

```python
class BSTIterator:

    def __init__(self, root: Optional[TreeNode]):
        self.my_stack = []
        while root:
            self.my_stack.append(root)
            root = root.left

    def next(self) -> int:
        curr = self.my_stack.pop()
        result = curr
        curr = curr.right
        while curr:
            self.my_stack.append(curr)
            curr = curr.left
        return result.val

    def hasNext(self) -> bool:
        return len(self.my_stack) > 0
```

## Initial Issues

- Used `stack.pop()` instead of `self.my_stack.pop()` — wrong variable name
- Returned the node object instead of `result.val`

## Key Learnings

- Controlled inorder traversal: push all left nodes, pop to get next, then go right and push lefts again
- This is essentially an iterative inorder traversal broken into steps

## Nuances to Remember

- **Amortized O(1) for `next()`**: Each node is pushed once and popped once across all calls — 2n total operations over n calls = O(1) average
- A "heavy" call that pushes many nodes means future calls will be cheap — it balances out
- **O(H) space**: Stack holds at most one root-to-leaf path
- `hasNext()` is O(1) — just check if stack is empty

## Q&A Highlights

- **Q: Why O(1) average if a single next() can push multiple nodes?** Every node is pushed/popped exactly once. 2n ops across n calls = O(1) amortized.
