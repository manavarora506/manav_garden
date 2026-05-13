# 78. Subsets

**Date:** 2026-03-17
**Difficulty:** Medium
**Topics:** Backtracking, Recursion, Array
**Link:** https://leetcode.com/problems/subsets/

## Final Solution

```python
class Solution:
    def subsets(self, nums: List[int]) -> List[List[int]]:
        result = []
        if not nums:
            return result
        
        def helper(index: int, curr: list[int]):
            if index == len(nums):
                result.append(curr.copy())
                return
            helper(index + 1, curr)
            curr.append(nums[index])
            helper(index + 1, curr)
            curr.pop()
        helper(0, [])
        return result
```

## Initial Issues

- Initialized `result = [[]]` which added a duplicate empty set — base case already handles the empty subset

## Key Learnings

### Include/Exclude Pattern
- At each index, make a binary decision: exclude the element or include it
- This creates a binary tree with `n` levels → `2^n` leaf nodes → `2^n` subsets
- Each leaf is reached by exactly one unique path, so **no duplicates possible**

### Backtracking
- `curr.append(nums[index])` — choose (include)
- `helper(index + 1, curr)` — explore
- `curr.pop()` — unchoose (undo the include so parent can explore exclude branch)
- Must use `curr.copy()` when appending to result — otherwise all entries point to same list

### Full Decision Tree for [1, 2, 3]

```
helper(0, [])
├── exclude 1: helper(1, [])
│   ├── exclude 2: helper(2, [])
│   │   ├── exclude 3 → [] ✓
│   │   ├── include 3 → [3] ✓
│   │   └── pop 3
│   ├── include 2: helper(2, [2])
│   │   ├── exclude 3 → [2] ✓
│   │   ├── include 3 → [2,3] ✓
│   │   └── pop 3
│   └── pop 2
├── include 1: helper(1, [1])
│   ├── exclude 2: helper(2, [1])
│   │   ├── exclude 3 → [1] ✓
│   │   ├── include 3 → [1,3] ✓
│   │   └── pop 3
│   ├── include 2: helper(2, [1,2])
│   │   ├── exclude 3 → [1,2] ✓
│   │   ├── include 3 → [1,2,3] ✓
│   │   └── pop 3
│   └── pop 2
└── pop 1

Result: [[], [3], [2], [2,3], [1], [1,3], [1,2], [1,2,3]]
```

## Nuances to Remember

- No duplicates because indices move strictly left to right — never revisit earlier indices
- Time: O(n * 2^n) — 2^n subsets, each up to length n to copy
- Space: O(n) for recursion depth + O(n * 2^n) for output
- `pop()` is critical — without it, the include decision leaks into the exclude branch
