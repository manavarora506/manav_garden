# Currency Exchange Rate Converter (Review Session)

**Date:** 2026-03-25
**Category:** API Design / Data Processing
**Parts Completed:** 4/4 (review)
**Language:** Python

## Problem Summary

Build a currency converter supporting direct rates, reverse rates, and transitive rates via graph traversal. Find the best (maximum) rate across all possible paths.

## Key Bugs Found on Review

### 1. defaultdict(float) instead of defaultdict(dict)
```python
# Bug: defaultdict(float) → self.lookup["USD"] = 0.0 → can't index into float
self.lookup = defaultdict(float)
# Fix: nested dict
self.lookup = defaultdict(dict)
```

### 2. DFS version bugs
```python
# Bug 1: max_rate used as both variable and function call
max_rate = max_rate(max_rate, currentRate)  # should be max()

# Bug 2: need nonlocal to modify outer variable
nonlocal max_rate

# Bug 3: set union with string
newVisited = visited | edge      # TypeError
newVisited = visited | {edge}    # correct

# Bug 4: passing edgeRate instead of newRate
newRate = edgeRate * currentRate
helper(edge, edgeRate, newVisited)  # should be newRate
```

## Complexity Discussion

### Why O(V!) not O(V+E)?
- Standard BFS with global visited → O(V+E) — each node processed once
- All-paths search with per-path visited → O(V!) — same node processed across many paths
- Per-path visited sets needed because different paths yield different rates

### Per-path visited vs backtracking
| Approach | Memory | Mechanism |
|----------|--------|-----------|
| New set per call: `visited \| {edge}` | O(V) per path | Immutable, allocates new set each branch |
| Backtracking: add/remove | O(V) total | One shared mutable set, undo after recursion |

### Optimization mention
- For large graphs: Bellman-Ford on log-transformed weights
- `log(a × b) = log(a) + log(b)` converts multiplication to addition → shortest path algorithms work

## Key Learnings

- Global visited = O(V+E). Per-path visited = O(V!). Same queue, fundamentally different algorithms
- defaultdict(float) vs defaultdict(dict) — the default value type matters
- Backtracking is more memory-efficient than copying sets per recursive call
- Always check: am I passing the accumulated value or the edge value?
