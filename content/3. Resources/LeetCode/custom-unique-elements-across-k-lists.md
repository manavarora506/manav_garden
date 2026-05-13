# Custom: Find Elements Unique to One List (k Lists)

**Date:** 2026-03-17
**Difficulty:** Medium
**Topics:** Hash Map, Set, MapReduce, Distributed Systems
**Link:** N/A (custom problem)

## Problem

Given k lists, find all elements that exist in exactly one list.

## Final Solution (Single Machine)

```python
from collections import defaultdict

class Solution:
    def __init__(self, total_lists):
        self.num_to_count = defaultdict(set)
        self.total_lists = total_lists

    def findUniqueElements(self) -> list:
        for id, curr_list in enumerate(self.total_lists):
            for num in curr_list:
                self.num_to_count[num].add(id)

        return [num for num in self.num_to_count if len(self.num_to_count[num]) == 1]
```

## Initial Issues

- First attempt used a set with add/remove — breaks with duplicates within a list (e.g., `[7, 7]` removes then re-adds)
- Used `defaultdict(list)` with `if id not in` check — O(n) lookup. Switched to `defaultdict(set)` for O(1)
- No need for `if` check with sets — `.add()` handles duplicates automatically
- `enumerate(len(...))` vs `enumerate(...)` — enumerate takes an iterable, not an int
- `for i, id in enumerate(...)` — `id` is the element not the index; use `for id, curr_list in enumerate(...)`

## Complexity

- Time: O(n * m) where n = num lists, m = length of longest list
- Space: O(n * m) worst case when every element is unique

## Key Learnings

### Core Insight
- Track **how many lists** a number appears in, not how many times it appears
- Use a set of list IDs per number — deduplicates within same list automatically

### MapReduce Extension (when data doesn't fit on one machine)

**Map:** each machine processes its list, emits `(num, list_id)` pairs (deduped per list)
**Shuffle:** framework hashes each key (`hash(num) % num_reducers`) to route all pairs with the same `num` to the same reducer
**Reduce:** merge list_ids into sets, output nums where set size == 1

### End-to-End Example

```
Machine 0: [1, 2, 4, 1]  →  emits (1,0), (2,0), (4,0)
Machine 1: [1, 2, 7]     →  emits (1,1), (2,1), (7,1)
Machine 2: [4, 7, 9]     →  emits (4,2), (7,2), (9,2)

Shuffle routes by hash(key) % num_reducers:
  Reducer A: 1→{0,1}, 2→{0,1}, 9→{2}
  Reducer B: 4→{0,2}, 7→{1,2}

Reduce (keep set size == 1):
  Reducer A: 9 ✓
  Reducer B: (none)

Result: [9]
```

Key: hash partitioning ensures all entries for the same num go to the same reducer. No single machine needs all the data.
