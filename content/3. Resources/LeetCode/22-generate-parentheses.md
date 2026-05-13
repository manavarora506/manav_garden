# 22. Generate Parentheses

**Date:** 2026-03-18
**Difficulty:** Medium
**Topics:** Backtracking, Recursion, String
**Link:** https://leetcode.com/problems/generate-parentheses/

## Final Solution

```python
class Solution:
    def generateParenthesis(self, n: int) -> List[str]:
        result = []

        def backtrack(curr: list, numOpen: int, numClosed: int):
            if numClosed > numOpen:
                return
            if numClosed > n or numOpen > n:
                return
            if len(curr) == n * 2:
                result.append("".join(curr.copy()))
                return
            curr.append("(")
            backtrack(curr, numOpen + 1, numClosed)
            curr.pop()
            curr.append(")")
            backtrack(curr, numOpen, numClosed + 1)
            curr.pop()

        backtrack(["("], 1, 0)
        return result
```

## Key Learnings

### Two choices at each position
- Append `(` or `)` — same include/exclude structure but with parentheses
- Two pruning conditions keep combinations valid:
  1. `numClosed > numOpen` — can't close more than opened
  2. `numOpen > n` or `numClosed > n` — can't exceed n of either type

### Why time is 2^(2n) not 2^n
- `n` means n **pairs**, so string length is `2n`
- At each of `2n` positions, 2 choices → 2^(2n) full tree before pruning
- Compare: subsets has `n` elements with include/exclude → 2^n
- Pruning cuts massively — for n=3: 2^6 = 64 possible → only 5 valid

### Exact complexity
- Valid combinations = nth Catalan number: C(n) = (2n)! / ((n+1)! * n!) ≈ 4^n / (n * sqrt(n))
- Time: O(n * C(n)) — C(n) valid strings, O(n) to copy each
- For interviews: "O(2^(2n) * n) upper bound, tightened by Catalan number" is fine

## Nuances to Remember

- Starting with `backtrack(["("], 1, 0)` is valid — every valid combination starts with `(`
- Every valid combination has exactly n `(` and n `)` characters
- `numClosed > numOpen` is the key validity check — ensures we never close an unopened paren
