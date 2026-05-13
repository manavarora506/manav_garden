# 119. Pascal's Triangle II

**Date:** 2026-03-16
**Difficulty:** Easy
**Topics:** Array, Dynamic Programming, Recursion
**Link:** https://leetcode.com/problems/pascals-triangle-ii/

## Final Solution

```python
class Solution:
    def getRow(self, rowIndex: int) -> List[int]:
        if rowIndex == 0:
            return [1]
        prevRow = self.getRow(rowIndex - 1)
        newRow = [1] * (rowIndex + 1)
        for i in range(1, rowIndex):
            newRow[i] = prevRow[i - 1] + prevRow[i]
        return newRow
```

## Initial Issues

- First attempt stored all rows (list of lists) — unnecessary since we only need the final row
- Treated return type inconsistently (flat list in base case, list of lists in recursive case)
- `return tree[rowIndex - 1]` worked by accident due to negative indexing cancellation
- Included unnecessary `rowIndex == 1` base case — `rowIndex == 0` alone is sufficient since `range(1, 1)` is empty

## Key Learnings

- Only need to track previous row to build the next — O(rowIndex) space optimization
- Recursive function returns a single row, not all rows — each frame builds next row from what it receives
- Fewer base cases = cleaner code. Always check if an extra base case is actually needed by tracing the next value through.

## Nuances to Remember

- `range(1, 1)` is empty — so row 1 `[1, 1]` builds correctly from row 0 `[1]` without special-casing
- Space: O(rowIndex) for the row + O(rowIndex) for call stack
- Time: O(rowIndex^2) — building each row takes O(row length)
# 119. Pascal's Triangle II

**Date:** 2026-03-17
**Difficulty:** Easy
**Topics:** Array, Dynamic Programming, Recursion
**Link:** https://leetcode.com/problems/pascals-triangle-ii/

## Final Solution

```python
class Solution:
    def getRow(self, rowIndex: int) -> List[int]:
        if rowIndex == 0:
            return [1]
        prevRow = self.getRow(rowIndex - 1)
        newRow = [1] * (rowIndex + 1)
        for i in range(1, rowIndex):
            newRow[i] = prevRow[i - 1] + prevRow[i]
        return newRow
```

## Initial Issues

- First attempt used a 2D array and helper function with `buildPascal` returning all rows — unnecessary since we only need the final row
- Had type mismatch: base cases returned flat lists but recursive case treated return value as list of lists
- Included unnecessary `rowIndex == 1` base case — `rowIndex == 0` is sufficient since `range(1, 1)` is empty

## Key Learnings

- Optimized from storing all rows to only keeping previous row — O(rowIndex) space
- Each frame only needs the previous row to build the next, so return a single list instead of list of lists
- `rowIndex == 0` returning `[1]` is the only base case needed — when `rowIndex == 1`, the loop `range(1, 1)` is empty so `newRow` stays as `[1, 1]`

## Nuances to Remember

- Row `i` has `i + 1` elements
- Middle elements: `prevRow[i-1] + prevRow[i]`
- Inner loop range is `range(1, rowIndex)` — skips first and last (always 1)
- Space: O(rowIndex) for the row + O(rowIndex) for call stack
