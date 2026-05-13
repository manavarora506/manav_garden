# Supply Chain Optimization

**Date:** 2026-02-09
**Category:** Data Processing / Dynamic Programming
**Parts Completed:** 3/4 (Part 4 discussed but not coded)
**Language:** Python

## Problem Summary

Select one factory per production stage to minimize total cost (building + transportation). Factories have a building cost and position along a railway. Transportation cost = absolute difference in positions between consecutive stage selections. Progressive parts: basic selection, add transport, N stages with DP, skip one stage.

## Solutions by Part

### Part 1: Basic Selection (No Transportation)

**Approach:** Greedy — pick the minimum building cost at each stage independently since there's no coupling between stages.

```python
def find_minimum_cost(stages):
    total_cost = 0
    for stage in stages:
        min_cost = min(stage, key=lambda x: x[0])
        total_cost += min_cost[0]
    return total_cost
```

**Time Complexity:** O(N × M)

### Part 2: With Transportation Cost (3 Stages)

**Approach:** Brute force — three nested loops to enumerate all 27 combinations (3³). Calculate building + transport cost for each and track the min.

```python
def find_minimum_cost(stages):
    total_cost = float('inf')
    for stage_0 in stages[0]:
        cost_0, position_0 = stage_0[0], stage_0[1]
        for stage_1 in stages[1]:
            cost_1, position_1 = stage_1[0], stage_1[1]
            for stage_2 in stages[2]:
                cost_2, position_2 = stage_2[0], stage_2[1]
                building_costs = cost_0 + cost_1 + cost_2
                transport_costs = abs(position_0 - position_1) + abs(position_1 - position_2)
                tc = building_costs + transport_costs
                total_cost = min(total_cost, tc)
    return total_cost
```

**Time Complexity:** O(M³)

### Part 3: N Stages (DP)

**Approach:** 2D DP where `dp[i][j]` = minimum total cost to complete stages 0 through i, ending with factory j at stage i. For each factory at stage i, try all factories from stage i-1 and pick the one minimizing cost. Only need to keep previous stage's DP row.

```python
def find_minimum_cost(stages):
    n = len(stages)
    if n == 0:
        return 0
    prev_dp = {}
    for j, factory in enumerate(stages[0]):
        prev_dp[(0, j)] = factory[0]

    for i in range(1, n):
        curr_dp = {}
        for j, curr_factory in enumerate(stages[i]):
            curr_cost, curr_pos = curr_factory[0], curr_factory[1]
            min_cost = float('inf')
            for k, prev_factory in enumerate(stages[i - 1]):
                prev_pos = prev_factory[1]
                transport = abs(curr_pos - prev_pos)
                total = prev_dp[(i-1, k)] + curr_cost + transport
                min_cost = min(min_cost, total)
            curr_dp[(i, j)] = min_cost
        prev_dp = curr_dp
    return min(prev_dp.values())
```

**Time Complexity:** O(N × M²)

### Part 4: Skip One Stage (Discussed, Not Coded)

**Approach:** For each stage index `skip` from 0 to N-1, create `stages[:skip] + stages[skip+1:]` and run Part 3 DP on the remaining N-1 stages. Return the minimum across all N skip choices.

**Time Complexity:** O(N² × M²)

## Edge Cases

- **Empty stages list** → return 0
- **Single stage** → just return min building cost, no transport
- **Skip first/last stage** → DP still works, just fewer stages
- **Only 2 stages, skip one** → single stage left, no transport cost
- **`math.abs` doesn't exist** → use builtin `abs()`

## Bugs & Issues

1. **Part 1:** `min()` with key returns the whole factory tuple, not just the cost — need `min_cost[0]`
2. **Part 2:** Used `math.abs` instead of `abs()`
3. **Part 2:** Variable name typo `building_cost` vs `building_costs`
4. **Part 2:** Initialized `total_cost = 0` instead of `float('inf')` — min would always return 0
5. **Part 3:** Base case stored entire factory instead of just cost (`factory` vs `factory[0]`)
6. **Part 3:** Used `prev_factory[i]` (stage index) instead of `prev_factory[1]` (position index)
7. **Part 3:** Forgot to initialize `curr_dp = {}` at start of outer loop
8. **Part 3:** Inconsistent naming `curr_dp` vs `cur_dp`

## Key Learnings

- **Greedy → brute force → DP** is a natural progression when choices become coupled across stages
- **2D DP state = (stage, choice)** when the cost of a choice depends on what you chose at the previous stage
- **Space optimization:** Only need the previous row of the DP table, not the entire table
- **Skip-one pattern:** Run the core algorithm N times, each time excluding one element — O(N) overhead factor

## Code Quality Notes

- Could simplify DP keys from `(i, j)` to just `j` since we only keep the previous row
- Unused variable `total_cost` in Part 3 — leftover from Part 2 structure
- Good progression from brute force to DP, but should practice coding the DP transition more fluently

## Q&A Highlights

- **What does `dp[i][j]` represent?** Min total cost through stages 0..i ending with factory j at stage i
- **Why can't we use greedy for Part 2+?** Transportation cost couples consecutive stage choices — locally optimal != globally optimal

---

## ⚠️ COME BACK TO THIS

**Need more 2D DP practice.** The transition from brute-force nested loops to DP was not yet fluent. Practice similar problems to build the muscle memory for:
- Identifying DP state dimensions
- Writing the recurrence relation
- Space-optimizing to only keep the previous row

### Similar LeetCode Problems (2D DP with sequential choices)

| Problem | Why It's Similar |
|---------|-----------------|
| **LC 256 - Paint House** | N houses, 3 colors, min cost with adjacent constraint — almost identical structure |
| **LC 265 - Paint House II** | Same as 256 but K colors — generalizes the M factories per stage |
| **LC 1289 - Minimum Falling Path Sum II** | Grid DP where each row choice depends on previous row — same transition pattern |
| **LC 931 - Minimum Falling Path Sum** | Simpler version of the falling path pattern |
| **LC 1143 - Longest Common Subsequence** | Classic 2D DP for building intuition |
| **LC 64 - Minimum Path Sum** | Grid DP with cumulative cost — good warmup |
| **LC 120 - Triangle** | Min path through triangle — similar "choose one per row" structure |

**Priority:** Start with LC 256 (Paint House) — it's nearly identical to this problem.
