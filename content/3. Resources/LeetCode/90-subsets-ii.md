# 90. Subsets II

**Date:** 2026-03-18
**Difficulty:** Medium
**Topics:** Backtracking, Recursion, Array, Sorting
**Link:** https://leetcode.com/problems/subsets-ii/

## Final Solution

```python
class Solution:
    def subsetsWithDup(self, nums: List[int]) -> List[List[int]]:
        result = []
        
        def backtrack(curr, index):
            if index == len(nums):
                result.append(curr.copy())
                return
            # exclude: skip all duplicates of nums[index]
            i = index + 1
            while i < len(nums) and nums[i] == nums[index]:
                i += 1
            backtrack(curr, i)
            # include
            curr.append(nums[index])
            backtrack(curr, index + 1)
            curr.pop()

        nums.sort()
        backtrack([], 0)
        return result
```

## Initial Issues

- First attempt used `if curr not in result` — works but O(n * 2^n) check per insert
- Tried skipping duplicates before the include/exclude decision — skips for both branches when it should only skip in the exclude branch
- Needed to sort first so duplicates are adjacent

## Key Learnings

### Duplicate Handling: Skip in Exclude Branch Only

- **Exclude** means "I don't want ANY of this value" → skip all copies
- **Include** means "I want at least one" → move to index+1 normally, can still include more copies

### Why Pruning Prevents Duplicates

For `nums = [1, 1, 2]`, without pruning there are two paths to `[1]`:
1. Exclude `1[0]`, include `1[1]` → `[1]`
2. Include `1[0]`, exclude `1[1]` → `[1]`

Pruning removes path 1: excluding `1[0]` skips past ALL `1`s, so you can never get `[1]` from that branch. `[1]` can only come from path 2.

### Visualization with pruning (`nums = [1, 1, 2]`)

```
                      []
                  /        \
          exclude 1[0]    include 1[0]
          SKIP to idx 2      [1]
             []              /     \
           /    \       ex 1[1]   inc 1[1]
         ex 2  inc 2      [1]      [1,1]
         []    [2]       / \       / \
                       ex  inc   ex  inc
                       [1] [1,2] [1,1] [1,1,2]

Result: [[], [2], [1], [1,2], [1,1], [1,1,2]] — no duplicates
```

### Same pattern used in Combination Sum II (40)

## Complexity

- Time: O(n * 2^n) — 2^n subsets worst case (all unique), O(n) to copy each. Pruning reduces work but doesn't change worst case
- Space: O(n * 2^n) for output; O(n) auxiliary (recursion depth + curr)
- Sorting: O(n log n) — dominated by backtracking
