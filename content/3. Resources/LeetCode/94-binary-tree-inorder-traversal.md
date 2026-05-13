# 94. Binary Tree Inorder Traversal

**Date:** 2026-02-02
**Difficulty:** Easy
**Topics:** Binary Tree, DFS, Stack
**Link:** https://leetcode.com/problems/binary-tree-inorder-traversal/

## Recursive Solution

```python
def inorderTraversal(self, root: Optional[TreeNode]) -> List[int]:
    result = []
    def dfs(node):
        if not node:
            return
        dfs(node.left)
        result.append(node.val)
        dfs(node.right)
        
    dfs(root)
    return result
```

## Iterative Solution (Stack)

```python
def inorderTraversal(self, root: Optional[TreeNode]) -> List[int]:
    result = []
    stack = []
    curr = root
    
    while stack or curr:
        # Go left as far as possible
        while curr:
            stack.append(curr)
            curr = curr.left
        
        # Pop, visit, go right
        curr = stack.pop()
        result.append(curr.val)
        curr = curr.right
    
    return result
```

## Initial Issues

1. **Class variable for result**: `result = []` at class level persists between test cases

2. **Wrong use of nonlocal**: `nonlocal` is for enclosing function scope, not class variables

3. **Calling wrong function**: Called `self.inorderTraversal()` inside `dfs()` instead of `dfs()`

4. **Confused BFS with iterative DFS**: Used queue and BFS pattern instead of stack

## Key Learnings

### When to Use `nonlocal`

| Operation | Need `nonlocal`? |
|-----------|-----------------|
| `result.append(x)` | No (modifying in place) |
| `result = result + [x]` | Yes (reassigning) |
| `count += 1` | Yes (reassigning) |

**Rule:** If `=` is on the left with the variable name → reassigning → needs `nonlocal`

### Iterative Inorder Pattern

```
while stack or curr:
    while curr:           # 1. Go left, pushing to stack
        stack.append(curr)
        curr = curr.left
    curr = stack.pop()    # 2. Pop and visit
    result.append(curr.val)
    curr = curr.right     # 3. Go right
```

The stack simulates the recursion call stack.

## Traversal Orders Reference

| Order | Pattern | Recursive |
|-------|---------|-----------|
| Inorder | Left → Root → Right | `dfs(left); visit; dfs(right)` |
| Preorder | Root → Left → Right | `visit; dfs(left); dfs(right)` |
| Postorder | Left → Right → Root | `dfs(left); dfs(right); visit` |
