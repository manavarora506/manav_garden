# 118. Pascal's Triangle

**Date:** 2026-03-16
**Difficulty:** Easy
**Topics:** Array, Dynamic Programming, Recursion
**Link:** https://leetcode.com/problems/pascals-triangle/

## Final Solution

### Iterative

```python
class Solution:
    def generate(self, numRows: int) -> List[List[int]]:
        result = [[1]]
        for i in range(1, numRows):
            row_arr = []
            prev_arr = result[i-1]
            for j in range(0, i+1):
                if j == 0 or j == i:
                    row_arr.append(1)
                else:
                    row_arr.append(prev_arr[j] + prev_arr[j-1])
            result.append(row_arr)
        return result
```

### Recursive

```python
class Solution:
    def generate(self, numRows: int) -> List[List[int]]:
        if numRows == 1:
            return [[1]]
        
        prevRows = self.generate(numRows - 1)
        newRow = [1] * numRows
        
        for i in range(1, numRows - 1):
            newRow[i] = prevRows[-1][i - 1] + prevRows[-1][i]
        
        prevRows.append(newRow)
        return prevRows
```

## Initial Issues

- Mixed up the middle element formula: used `prev[j] + prev[j+1]` instead of `prev[j-1] + prev[j]`
- Included unnecessary `numRows == 0` base case in recursive version — `generate(1)` never calls `generate(0)`

## Key Learnings

- Each row starts and ends with `1`, middle elements are sum of two adjacent elements from previous row: `prev[j-1] + prev[j]`
- `prevRows[-1]` accesses the last element of a list (negative indexing in Python)
- Recursive approach: recurse down to base case `numRows == 1`, then build each row on the way back up — same "bucket brigade" pattern as BST search and reverse linked list

## Nuances to Remember

- Row `i` has `i+1` elements (row 0 has 1, row 1 has 2, etc.)
- Inner loop range is `range(1, numRows - 1)` to skip first and last elements (always 1)
- `numRows == 0` base case is dead code — only need `numRows == 1`
- Recursive call stack trace for `generate(3)`:
  1. `generate(3)` → calls `generate(2)`, waits
  2. `generate(2)` → calls `generate(1)`, waits
  3. `generate(1)` → returns `[[1]]`
  4. Unwind: `generate(2)` builds `[1,1]`, returns `[[1],[1,1]]`
  5. Unwind: `generate(3)` builds `[1,2,1]`, returns `[[1],[1,1],[1,2,1]]`
