# 116. Populating Next Right Pointers in Each Node

**Date:** 2026-01-30
**Difficulty:** Medium
**Topics:** BFS, Level Order Traversal, Binary Tree, Linked List
**Link:** https://leetcode.com/problems/populating-next-right-pointers-in-each-node/

## Final Solution (BFS with Queue)

```python
class Solution:
    def connect(self, root: 'Optional[Node]') -> 'Optional[Node]':
        if not root:
            return None
        queue = deque()
        queue.append(root)
        while queue:
            size = len(queue)
            for i in range(size):
                curr = queue.popleft()
                if i == size - 1:
                    curr.next = None
                else:
                    nextNode = queue[0]
                    curr.next = nextNode
                if curr.left:
                    queue.append(curr.left)
                if curr.right:
                    queue.append(curr.right)
        return root
```

## O(1) Space Solution (Using Established Next Pointers)

```python
class Solution:
    def connect(self, root: 'Optional[Node]') -> 'Optional[Node]':
        if not root:
            return None
        leftmost = root
        while leftmost.left:           # while there's a level below
            curr = leftmost
            while curr:                # traverse current level
                curr.left.next = curr.right
                curr.right.next = curr.next.left if curr.next else None
                curr = curr.next       # move right across level
            leftmost = leftmost.left   # move down to next level
        return root
```

## Initial Issues

- **Wrong pointer name:** Used `curr.right = None` instead of `curr.next = None`
- **Undefined variable:** Used `node.right` where `node` was never defined (meant `curr`)
- **Wrong "next" logic:** Initially tried to use `curr.right` to find the next node in level, but the next node is at `queue[0]` (front of queue after popping)

## Key Learnings

- **BFS level tracking:** Use `size = len(queue)` at the start of each level to know how many nodes belong to the current level
- **Peeking the queue:** After popping `curr`, `queue[0]` is the next node in the same level (before adding children)
- **Timing matters:** Peek at `queue[0]` BEFORE adding curr's children to get the right node
- **O(1) space insight:** Once level N is connected via `next` pointers, you can traverse it like a linked list to connect level N+1

## Nuances to Remember

- Perfect binary tree = all leaves at same depth, every parent has 2 children
- For O(1) space: use two loops
  - Outer loop: moves down levels (leftmost = leftmost.left)
  - Inner loop: moves across level (curr = curr.next)
- Connecting children from curr:
  - Siblings: `curr.left.next = curr.right`
  - Cousins: `curr.right.next = curr.next.left if curr.next else None`

## Q&A Highlights

- **Q: How do you get the next node in BFS without extra variables?**  
  A: After `popleft()`, `queue[0]` is the next node in the same level (since children aren't added yet)

- **Q: How does O(1) space work?**  
  A: The `next` pointers you build at level N form a linked list. Traverse that linked list to connect level N+1. No queue needed.

## Pattern Recognition

- Level-order traversal with `size = len(queue)` is a common BFS pattern
- "Use what you've already built" - the O(1) solution reuses the `next` pointers as a traversal mechanism


## Struggle Points (To Revisit)

- **Connecting cousins in O(1) space:** Had trouble with `curr.right.next = curr.next.left` - the logic of reaching across to the "cousin" node via `curr.next`
- **Two-loop structure:** The pattern of outer loop (down levels) + inner loop (across level) for O(1) space wasn't intuitive yet
- **Did not complete the O(1) solution** - come back and implement it fully
