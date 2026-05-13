# 46. Permutations

**Date:** 2026-03-18
**Difficulty:** Medium
**Topics:** Backtracking, Recursion, Array
**Link:** https://leetcode.com/problems/permutations/

## Final Solution

```python
class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        result = []
        def backtrack(curr: list[int], seen: set):
            if len(curr) == len(nums):
                result.append(curr.copy())
                return
            for num in nums:
                if num not in seen:
                    curr.append(num)
                    seen.add(num)
                    backtrack(curr, seen)
                    curr.pop()
                    seen.remove(num)
        backtrack([], set())
        return result
```

## Initial Issues

- Tried include/exclude pattern from subsets — doesn't work for permutations because order matters (`[1,2]` and `[2,1]` are different permutations but same subset)
- Include/exclude moves left to right through indices, but permutations need to pick any unused element at any position
- Put `pop()` outside the for loop instead of inside — must undo each choice before trying the next
- Used `seen.pop()` instead of `seen.remove(num)` — sets don't have positional pop

## Key Learnings

### Permutations vs Subsets/Combinations

| | Subsets/Combinations | Permutations |
|---|---|---|
| Pattern | Include/exclude at each index | Loop all elements, pick unused |
| Order | Doesn't matter | Matters |
| Traversal | Left to right through indices | Any unused element at each position |
| Tracking | Index moves forward | `seen` set |

### How it works
- At each position, loop through **all** nums
- Skip any already in `seen` set
- Add to `curr` and `seen`, recurse, then undo both (backtrack)
- No index needed — the `seen` set controls which elements are available

### Backtracking must be inside the loop
```python
for num in nums:
    if num not in seen:
        curr.append(num)      # choose
        seen.add(num)
        backtrack(curr, seen)  # explore
        curr.pop()             # unchoose
        seen.remove(num)       # unchoose
```
Each choice must be undone **before** trying the next element in the loop.

## Complexity

- Time: O(n * n!) — n! permutations, O(n) to copy each
- Space: O(n * n!) for output; O(n) auxiliary (recursion depth + curr + seen)
- First position has n choices, second has n-1, third has n-2... = n!
