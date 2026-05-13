# 17. Letter Combinations of a Phone Number

**Date:** 2026-03-19
**Difficulty:** Medium
**Topics:** Backtracking, String, Hash Map
**Link:** https://leetcode.com/problems/letter-combinations-of-a-phone-number/

## Final Solution

```python
class Solution:
    def letterCombinations(self, digits: str) -> List[str]:
        digit_to_letters = {1: "", 2: "abc", 3: "def", 4: "ghi", 5: "jkl", 6: "mno", 7: "pqrs", 8: "tuv", 9: "wxyz"}
        result = []

        def backtrack(curr: list, index: int):
            if len(curr) == len(digits):
                result.append("".join(curr.copy()))
                return
            digit = int(digits[index])
            letters = digit_to_letters[digit]
            for letter in letters:
                curr.append(letter)
                backtrack(curr, index + 1)
                curr.pop()
        
        backtrack([], 0)
        return result
```

## Key Learnings

- Each digit maps to 3-4 letters — loop through all letters for current digit, recurse on next digit
- Base case: `curr` length == `digits` length — we've assigned a letter to each digit
- Standard backtracking: append → recurse → pop

## Complexity (using choices^depth framework)

- **Choices per step:** up to 4 letters per digit (7→"pqrs", 9→"wxyz")
- **Depth:** n = length of `digits` string

### Time: O(n * 4^n)
- 4^n combinations worst case
- O(n) to copy/join each combination

### Space:
- **Auxiliary (not counting output):** O(n) — recursion depth + curr
- **Total (counting output):** O(n * 4^n) — up to 4^n strings of length n stored in result
- In interviews, space usually means auxiliary = O(n)
