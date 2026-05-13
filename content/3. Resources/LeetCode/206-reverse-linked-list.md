# 206. Reverse Linked List

**Date:** 2026-03-16
**Difficulty:** Easy
**Topics:** Linked List, Recursion
**Link:** https://leetcode.com/problems/reverse-linked-list/

## Final Solution

```python
class Solution:
    def reverseList(self, head: Optional[ListNode]) -> Optional[ListNode]:
        if not head or not head.next:
            return head
        newHead = self.reverseList(head.next)
        head.next.next = head
        head.next = None
        return newHead
```

## Initial Issues

- Struggled significantly with the recursive approach
- Specifically confused by the pointer manipulation: `head.next.next = head` and `head.next = None`
- Hard to visualize what's happening as recursion unwinds

## Key Learnings

- The recursive solution works by recursing to the end of the list first, then reversing pointers on the way back up the call stack
- `newHead` is always the last node (found at base case) and gets passed all the way back unchanged
- The key insight: when recursion unwinds at node `head`, `head.next` still points to the next node. So `head.next.next = head` makes that next node point back to `head` (reversing the link)
- `head.next = None` breaks the old forward link to avoid cycles

## Nuances to Remember

- Walk through example: `1->2->3->None`
  - Base case returns `3` as `newHead`
  - At `head=2`: `head.next` is `3`, so `3.next = 2` (now `3->2`), then `2.next = None`
  - At `head=1`: `head.next` is `2`, so `2.next = 1` (now `2->1`), then `1.next = None`
  - Result: `3->2->1->None`, `newHead` is still `3`
- The iterative approach (prev/curr/next pointers) is more intuitive but recursive is important to understand for interview follow-ups
- `head.next.next` is NOT double-jumping — it's accessing the `next` pointer OF the node that `head.next` points to
