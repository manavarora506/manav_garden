# 74. Search a 2D Matrix

**Date:** 2026-03-19
**Difficulty:** Medium
**Topics:** Binary Search, Matrix
**Link:** https://leetcode.com/problems/search-a-2d-matrix/

## Final Solution

```python
class Solution:
    def searchMatrix(self, matrix: List[List[int]], target: int) -> bool:
        rows = len(matrix)
        cols = len(matrix[0])
        start = 0
        end = rows * cols - 1

        while start <= end:
            mid = start + (end - start) // 2
            row = mid // cols
            col = mid % cols
            if matrix[row][col] == target:
                return True
            elif matrix[row][col] > target:
                end = mid - 1
            else:
                start = mid + 1
        return False
```

## Key Learnings

### Treat 2D matrix as virtual 1D sorted array
- Don't flatten it — just convert flat index to row/col on the fly
- `end = rows * cols - 1` (total elements minus 1)

### Flat index to row/col conversion
- `row = mid // cols` — how many full rows of `cols` elements can I skip?
- `col = mid % cols` — how many leftover positions after those full rows?
- Analogy: like converting minutes to hours (150 min → 2 hrs, 30 min with "row" size 60)

### Example (3x4 matrix)
```
Index:  0  1  2  3
        4  5  6  7
        8  9  10 11

mid = 7 → row = 7//4 = 1, col = 7%4 = 3
mid = 10 → row = 10//4 = 2, col = 10%4 = 2
```

## Complexity

- Time: O(log(m * n)) — binary search over m*n elements
- Space: O(1) — no extra data structures
