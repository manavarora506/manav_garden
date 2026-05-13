# Minimum Penalty for a Shop

**Date:** 2026-02-22
**Category:** Data Processing
**Parts Completed:** 1/1
**Language:** Python
**Status:** COME BACK TO THIS — need more practice with prefix sum / incremental adjustment pattern

## Problem Summary

Given a string of 'Y' and 'N' representing customer visits each hour, find the earliest closing hour that minimizes penalty. Penalty = +1 for each open hour with no customers + each closed hour with customers. LeetCode 2483.

## Solution

**Approach:** Calculate penalty for closing at hour 0 (all Y's count as penalty). Then incrementally adjust: moving closing time forward by 1 means one hour switches from closed→open. If that hour is Y, penalty -1; if N, penalty +1. Track running total and minimum separately.

```python
class Solution:
    def bestClosingTime(self, customers: str) -> int:
        bestClosingTime = 0
        minPenalty = sum(customer == "Y" for customer in customers)
        currPenalty = minPenalty
        for hour in range(1, len(customers) + 1):
            if customers[hour - 1] == 'Y':
                currPenalty -= 1
            elif customers[hour - 1] == 'N':
                currPenalty += 1
            if currPenalty < minPenalty:
                minPenalty = currPenalty
                bestClosingTime = hour
        return bestClosingTime
```

## Edge Cases

- Closing at hour 0 (shop never opens) — penalty is count of all Y's
- Closing at hour n (shop open all day) — penalty is count of all N's
- Tie in minimum penalty — return earliest hour (handled by `<` not `<=`)
- All Y's — best to close at end
- All N's — best to close at 0

## Bugs & Issues

1. **Nested loop instead of single pass** — Initially wrote an inner loop iterating over hours 0 to j for each closing time j, defeating the O(n) optimization. Needed to understand that only one hour changes per step.
2. **Checking `customers[hour]` instead of `customers[hour - 1]`** — Off-by-one: when closing at hour j, it's hour j-1 that switched status.
3. **Resetting `currPenalty = minPenalty` inside the loop** — Lost track of the running total by resetting to the best-seen value each iteration. Needed two separate variables: running total vs best seen.
4. **Modifying `minPenalty` as both running total and minimum** — Used one variable for two purposes. Fixed by introducing `currPenalty` as the running total outside the loop.
5. **Confusion between `count()` and `sum()`** — `count` isn't a standalone function for generator expressions; `sum()` with boolean expression works.

## Key Learnings

- **Prefix sum / incremental adjustment pattern** — Instead of O(n²) recalculation, figure out how moving one step changes the answer → O(n)
- **Two variables, not one** — Separate the running total (`currPenalty`) from the best seen (`minPenalty`). This is a universal pattern for "find minimum/maximum over a sequence of incremental changes"
- **Off-by-one discipline** — When closing at hour j, hour j-1 is what changed. Always be precise about which index you're adjusting.
- **Stripe relevance** — This pattern applies to batch-processing optimization, e.g., "what's the optimal time to cut off a batch to minimize cost?"

## Code Quality Notes

- Clean O(n) single-pass solution
- Clear variable names distinguish running state from best-so-far
- `sum(c == "Y" for c in customers)` is idiomatic Python for counting matches

## Q&A Highlights

- **Q: Why two variables?** A: `currPenalty` is like the current temperature — goes up and down. `minPenalty` is the lowest recorded — only decreases. Resetting current to lowest each step loses track of where you actually are.
- **Q: Why `hour - 1`?** A: Closing at hour j means hours 0..j-1 are open. When moving from j-1 to j, hour j-1 switches from closed to open — that's the one character you check.
