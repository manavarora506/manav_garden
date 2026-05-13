# 39. Combination Sum

**Date:** 2026-03-17
**Difficulty:** Medium
**Topics:** Backtracking, Recursion, Array
**Link:** https://leetcode.com/problems/combination-sum/

## Final Solution

```python
class Solution:
    def combinationSum(self, candidates: List[int], target: int) -> List[List[int]]:
        result = []
        
        def helper(curr: list[int], currSum: int, index: int):
            if currSum == target:
                result.append(curr.copy())
                return
            if currSum > target or index >= len(candidates):
                return
            helper(curr, currSum, index + 1)
            curr.append(candidates[index])
            currSum += candidates[index]
            helper(curr, currSum, index)
            curr.pop()
        
        helper([], 0, 0)
        return result
```

## Initial Issues

- Forgot `return` after appending valid result — kept exploring past the target
- Missing `index >= len(candidates)` bounds check — caused index out of bounds
- Tried appending two elements before recursing without popping between — overcomplicated the include/exclude pattern
- Needed to return to simple subsets-style include/exclude structure

## Key Learnings

### Same include/exclude pattern as Subsets, with two twists:
1. **Include** stays at same index (can reuse candidate)
2. **Exclude** moves to next index (done with this candidate forever)
3. `currSum > target` prunes branches early

### Mental Model: "How many of each?"
Think of it like filling a shopping cart candidate by candidate:
- "How many 2s do I use?" → include keeps adding 2s, exclude moves to next
- "How many 3s do I use?" → same process
- Each unique path through the tree = a unique count of each candidate

### Trace for candidates=[2,3], target=6

```
helper([], 0, index=0)  → "How many 2s?"

  exclude 2 (index=1)   → "Zero 2s. How many 3s?"
    include 3 → include 3 → [3,3]=6 ✓

  include 2 (index=0)   → "At least one 2. More?"
    exclude 2 (index=1) → "Exactly one 2. How many 3s?"
      include 3 → [2,3,3]=8 → too big

    include 2 (index=0) → "At least two 2s. More?"
      exclude 2 (index=1) → "Exactly two 2s. How many 3s?"
        include 3 → [2,2,3]=7 → too big

      include 2 → [2,2,2]=6 ✓

Result: [[3,3], [2,2,2]]
```

## Nuances to Remember

- Exclude = "I'm done deciding how many of this candidate, move to next"
- Include = "Use at least one more of this candidate"
- `currSum > target` is critical — without it, include branch loops forever
- Time: O(n^(T/M)) where T=target, M=min candidate — branching factor at each level
- Same backtracking structure as subsets: append → recurse → pop
