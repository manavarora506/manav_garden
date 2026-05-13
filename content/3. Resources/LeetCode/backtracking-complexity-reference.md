# Backtracking Time & Space Complexity Reference

**Date:** 2026-03-18
**Topics:** Backtracking, Complexity Analysis

## Framework

For any backtracking problem, ask two questions:

1. **How many choices at each step?**
2. **How many steps (depth)?**

**Time = choices^depth** (before pruning), multiplied by cost per leaf (usually O(n) to copy result)

**Space = O(depth)** for recursion stack, plus O(output size) if storing results

## Cheat Sheet

| Problem | Choices per step | Depth | Time | Space |
|---|---|---|---|---|
| Subsets (78) | 2 (include/exclude) | n elements | O(n * 2^n) | O(n) |
| Subsets II (90) | 2 (with pruning) | n elements | O(n * 2^n) | O(n) |
| Permutations (46) | n, n-1, n-2... | n positions | O(n * n!) | O(n) |
| Parentheses (22) | 2 (open/close) | 2n positions | O(n * 2^(2n)) → Catalan | O(n) |
| Combination Sum (39) | 2 (include/exclude) | up to target/min | O(n * 2^(T/M)) | O(T/M) |
| Word Search (79) | 4 (directions) | L (word length) | O(m*n * 4^L) | O(L) |

## Key: What Counts as a "Step"?

This is where most confusion comes from:

- **Subsets/Combinations:** each **element** is a step — include or exclude it
- **Permutations:** each **position** is a step — which unused element goes here?
- **Parentheses:** each **character position** is a step — there are 2n positions, NOT n
- **Word Search:** each **letter in the word** is a step — explore 4 directions per letter

## Why Permutations are n! not 2^n

- Subsets: n decisions, each binary → 2^n
- Permutations: n decisions, but choices **shrink** → n * (n-1) * (n-2) * ... * 1 = n!

## Pruning Reduces Actual Runtime

The formulas above are **worst case before pruning**. In practice:
- Combination Sum prunes when sum > target
- Parentheses prunes when closed > open or either > n
- Word Search prunes on mismatch, out of bounds, visited
- Subsets II prunes duplicate branches

Pruning doesn't change the worst-case big-O (except Catalan for parentheses) but dramatically reduces actual runtime.
