# 144. Binary Tree Preorder Traversal

**Date:** 2026-02-02
**Difficulty:** Easy
**Topics:** Binary Tree, DFS, Stack
**Link:** https://leetcode.com/problems/binary-tree-preorder-traversal/

## Recursive Solution

```python
def preorderTraversal(self, root: Optional[TreeNode]) -> List[int]:
    result = []
    if not root:
        return []
    def dfs(node: TreeNode):
        if not node:
            return 
        result.append(node.val)
        dfs(node.left)
        dfs(node.right)
    dfs(root)
    return result
```

## Iterative Solution (Stack)

```python
def preorderTraversal(self, root: Optional[TreeNode]) -> List[int]:
    if not root:
        return []
    result = []
    stack = [root]
    
    while stack:
        node = stack.pop()
        result.append(node.val)
        if node.right:  # Push right first
            stack.append(node.right)
        if node.left:   # Then left (popped first)
            stack.append(node.left)
    
    return result
```

## Key Learnings

### Preorder is Simpler Iteratively Than Inorder

- **Preorder:** Visit immediately when popped, then push children
- **Inorder:** Must go all the way left first, then visit on the way back up

### Why Push Right Before Left?

Stack is LIFO. To get left-to-right order:
```
Push right → Push left → Pop left first → Pop right second
```

### Iterative Comparison

| Traversal | Iterative Pattern |
|-----------|-------------------|
| Preorder | Pop, visit, push right, push left |
| Inorder | Go left pushing all, pop and visit, go right |
| Postorder | More complex (need to track visited) |
