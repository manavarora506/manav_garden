# 79. Word Search

**Date:** 2026-03-18
**Difficulty:** Medium
**Topics:** Backtracking, Matrix, DFS
**Link:** https://leetcode.com/problems/word-search/

## Final Solution

```python
class Solution:
    def exist(self, board: List[List[str]], word: str) -> bool:
        rows = len(board)
        cols = len(board[0])

        def backtrack(row: int, col: int, index: int):
            if index == len(word):
                return True
            if row < 0 or row > rows - 1 or col < 0 or col > cols - 1 or board[row][col] == '.' or board[row][col] != word[index]:
                return False

            directions = [(0, 1), (0, -1), (1, 0), (-1, 0)]

            prev = board[row][col]
            board[row][col] = '.'

            for dx, dy in directions:
                if backtrack(row + dx, col + dy, index + 1):
                    return True

            board[row][col] = prev
            return False

        for row in range(rows):
            for col in range(cols):
                if board[row][col] == word[0]:
                    if backtrack(row, col, 0):
                        return True
        return False
```

## Initial Issues

- **Saved `prev` after marking `'.'`:** `prev = board[row][col]` was after `board[row][col] = '.'`, so prev was always `'.'` — restore did nothing
- **Restored cell inside the for loop:** `board[row][col] = prev` inside the loop meant the cell was unvisited between directions, allowing the current cell to be revisited. Cell must stay `'.'` while trying all 4 directions — restore only after the loop

## Key Learnings

### Mark/restore pattern for grid backtracking
```python
prev = board[row][col]      # save BEFORE marking
board[row][col] = '.'       # mark visited

for dx, dy in directions:   # try all directions
    if backtrack(...):
        return True

board[row][col] = prev      # restore AFTER all directions tried
```

- Save before marking, restore after loop — NOT inside the loop
- Cell must stay marked while exploring all directions to prevent revisiting

### Why mutate board instead of using a visited set
- Mutating board is O(1) space vs O(n) for a visited set
- Use `'.'` as sentinel value, check for it in the boundary condition

## Complexity

### Time: O(m * n * 4^L)
- m * n starting cells
- From each cell, 4 directions at each level, L levels deep (L = word length)
- Visualization for word "ABC":
  ```
  Start (1,1) 'A'
  ├── 4 branches for 'B'
  │   ├── 4 branches each for 'C'  = 4*4 = 16 leaves
  ```
- Most branches pruned early (no match, out of bounds, visited)
- In practice closer to 3^L since you can't go back the way you came

### Space: O(L)
- Recursion depth = word length L
- No extra data structures (mutate board in-place)
