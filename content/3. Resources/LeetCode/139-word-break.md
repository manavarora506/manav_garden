# 139. Word Break

**Date:** 2026-03-16
**Difficulty:** Medium
**Topics:** Recursion, Memoization, Dynamic Programming, String
**Link:** https://leetcode.com/problems/word-break/

## Final Solution

```python
class Solution:
    def __init__(self):
        self.my_set = set()

    def wordBreak(self, s: str, wordDict: List[str]) -> bool:
        if s == "":
            return True
        for word in wordDict:
            if s.startswith(word):
                remaining = s[len(word):]
                if remaining not in self.my_set:
                    if self.wordBreak(remaining, wordDict):
                        return True
        self.my_set.add(s)
        return False
```

## Initial Issues

- First attempt tried to find and remove words from anywhere in the string (using `find` and splice) — wrong because word break requires contiguous segments from left to right
- Used `word in s` instead of `s.startswith(word)` — matched words in the middle of the string
- Returned immediately on first matching word without trying alternatives — need to try other words if one path fails
- Forgot to use return value of recursive call (same "bucket brigade" mistake from BST search)
- Initially placed memoization cache add before knowing the result — should only cache a string as "failed" after all words have been tried against it

## Key Learnings

- **Core insight:** at each step, decide how much of the string to consume from the front. If a word matches the start, recurse on the remainder `s[len(word):]`
- **Backtracking structure:** try each word, if recursive call returns True return True, otherwise let the loop continue. Return False after all words exhausted.
- **Memoization:** cache strings that returned False (after trying all words) so we don't re-explore them. Add to cache AFTER the for loop, not before recursing.
- Only need to cache failures — successes return True immediately up the call stack

## Nuances to Remember

- `s.startswith(word)` ensures we only consume from the front — this is what makes it a valid word break
- Remaining string is simply `s[len(word):]` — no need for find/splice logic
- There are only `n` unique suffixes of a string, so at most `n` subproblems
- **Time complexity:** O(n * m * k) with memoization, where n = string length, m = dict size, k = avg word length. Without memo: O(m^n) exponential
- **Space:** O(n) for cache + O(n) call stack

## Q&A Highlights

- **Why cache after the loop, not before recursing?** If you cache before recursing, you mark a string as "seen" before knowing if it succeeds or fails. A string should only be marked as failed after ALL words have been tried against it.
- **Why `startswith` not `in`?** Word break splits the string into contiguous left-to-right segments. Finding a word in the middle and removing it breaks this constraint.
# 139. Word Break

**Date:** 2026-03-17
**Difficulty:** Medium
**Topics:** Recursion, Memoization, String
**Link:** https://leetcode.com/problems/word-break/

## Final Solution

```python
class Solution:
    def __init__(self):
        self.my_set = set()

    def wordBreak(self, s: str, wordDict: List[str]) -> bool:
        if s == "":
            return True
        for word in wordDict:
            if s.startswith(word):
                remaining = s[len(word):]
                if remaining not in self.my_set:
                    if self.wordBreak(remaining, wordDict):
                        return True
        self.my_set.add(s)
        return False
```

## Initial Issues

- First attempt tried to find and remove words from anywhere in the string (`word in s` + `find` + splice). Word break requires consuming **contiguous segments from left to right**, not removing from arbitrary positions
- Used `return` on first matching word instead of trying other words when recursion fails
- Forgot to use the return value of recursive call (same "bucket brigade" issue as BST search and reverse linked list)

## Key Learnings

- **Core idea:** at each step, try every word that matches the **start** of `s`. If it matches, recurse on the remainder `s[len(word):]`
- `s.startswith(word)` ensures left-to-right consumption
- If recursive call returns `False`, let the for loop continue to try other words
- If no word works, return `False`

### Memoization
- Without cache: exponential O(m^n) — multiple paths can lead to the same remaining substring
- Cache **failed** substrings only — add `s` to set **after** the for loop (all words tried, none worked)
- Only cache after exhausting all options, not before recursing
- Key insight: there are only `n` unique suffixes of a string, so at most `n` subproblems
- With cache: O(n * m * k) where n = string length, m = dict size, k = avg word length

## Nuances to Remember

- The set stores strings that are **confirmed failures** — only added after all words have been tried
- `startswith` + `s[len(word):]` is the clean pattern for consuming from the front
- Don't return immediately on first `startswith` match — the recursive path may fail and you need to try other words
