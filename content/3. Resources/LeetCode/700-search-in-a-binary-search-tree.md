# 700. Search in a Binary Search Tree

**Date:** 2026-02-17
**Difficulty:** Easy
**Topics:** BST, Binary Search
**Link:** https://leetcode.com/problems/search-in-a-binary-search-tree/

## Final Solution

### Iterative

```python
def searchBST(self, root: Optional[TreeNode], val: int) -> Optional[TreeNode]:
    while root:
        if root.val == val:
            return root
        elif root.val > val:
            root = root.left
        else:
            root = root.right
    return root
```

### Recursive

```python
def searchBST(self, root: Optional[TreeNode], val: int) -> Optional[TreeNode]:
    if not root:
        return None
    if root.val == val:
        return root
    elif root.val > val:
        return self.searchBST(root.left, val)
    else:
        return self.searchBST(root.right, val)
```

## Key Learnings

- Straightforward BST search — go left if val is smaller, right if larger
- Iterative is O(H) time O(1) space; recursive is O(H) time O(H) space due to call stack

## Nuances to Remember

- Iterative preferred for O(1) space
- Return the subtree rooted at the found node, not just the value

## Session 2 — Recursion Deep Dive (2026-03-16)

### Why `return` is needed at every recursive call

- Each frame is its own function call with its own scope
- When `searchBST(3, 3)` finds the answer and returns it, that value only goes to its **immediate caller** (e.g., frame for node 2), not all the way to the top
- Every frame must explicitly `return` the result to pass it up one level — like a **bucket brigade**
- Without `return` before the recursive call, the result is received but thrown away, and the frame implicitly returns `None`
- This is the same pattern as 206 Reverse Linked List where `newHead` had to be returned at every frame

### Call stack trace example

Tree: `[4, 2, 7, 1, 3]`, val = 3:
1. `searchBST(4, 3)` → 4 > 3, calls `searchBST(2, 3)` ... waiting
2. `searchBST(2, 3)` → 2 < 3, calls `searchBST(3, 3)` ... waiting
3. `searchBST(3, 3)` → found! Returns node 3
4. Unwind: frame 2 returns node 3 → frame 1 returns node 3 → caller gets node 3

Answer passes through **3 returns**, not just 1.

### Complexity

|  | Time (avg) | Time (worst) | Space |
|---|---|---|---|
| Recursive | O(log N) | O(N) — skewed tree | O(N) — call stack |
| Iterative | O(log N) | O(N) — skewed tree | O(1) — single pointer |

### When to use which

- **Iterative** is the better default for BST search — equally readable, O(1) space
- **Recursive** is fine when tree is balanced and you want clean code, but call stack costs O(H) space
