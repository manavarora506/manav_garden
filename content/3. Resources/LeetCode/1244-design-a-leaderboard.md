# 1244. Design a Leaderboard

**Date:** 2026-03-17
**Difficulty:** Medium
**Topics:** Hash Map, Heap, Sorting, Design
**Link:** https://leetcode.com/problems/design-a-leaderboard/

## Final Solution

### Simple Sorting Approach (Preferred)

```python
from collections import defaultdict

class Leaderboard:
    def __init__(self):
        self.player_to_score = defaultdict(int)

    def addScore(self, playerId: int, score: int) -> None:
        self.player_to_score[playerId] += score

    def top(self, K: int) -> int:
        return sum(sorted(self.player_to_score.values(), reverse=True)[:K])

    def reset(self, playerId: int) -> None:
        self.player_to_score[playerId] = 0
```

### Lazy Deletion Heap Approach

```python
from collections import defaultdict
import heapq

class Leaderboard:
    def __init__(self):
        self.player_to_score = defaultdict(int)
        self.maxHeap = []

    def addScore(self, playerId: int, score: int) -> None:
        self.player_to_score[playerId] += score
        new_score = self.player_to_score.get(playerId)
        heapq.heappush(self.maxHeap, (-new_score, playerId))

    def top(self, K: int) -> int:
        temp = self.maxHeap.copy()
        totalSum = 0
        popped = 0
        seen = set()
        while temp and popped < K:
            score, playerId = heapq.heappop(temp)
            if self.player_to_score[playerId] != -score or playerId in seen:
                continue
            seen.add(playerId)
            totalSum += -score
            popped += 1
        return totalSum

    def reset(self, playerId: int) -> None:
        self.player_to_score[playerId] = 0
```

## Initial Issues

- `addScore` replaced score (`= score`) instead of accumulating (`+= score`)
- Pushed individual increment onto heap instead of player's **total** score — stale check wouldn't match
- Didn't track seen players in `top` — same player counted multiple times when they had multiple valid-looking heap entries with same score

## Key Learnings

### Lazy Deletion Pattern
- Don't remove stale entries from heap immediately — skip them when popping
- Validate by checking if popped score matches current score in dict
- Need a `seen` set to avoid counting same player twice (can have duplicate valid scores after add+reset+add)

### Approach Tradeoffs

| Approach | addScore | top | reset | Notes |
|---|---|---|---|---|
| Sort values | O(1) | O(n log n) | O(1) | Simple, no stale data, n = num players |
| Lazy heap | O(log n) | O(n log n) worst | O(1) | Heap grows with stale entries, n = total heap size |
| SortedList | O(log n) | O(K) | O(log n) | Best but requires `sortedcontainers` library |

### Why not a min heap of size K?
- Works for static data (like 973 - K Closest Points)
- Fails here because scores **change** — can't efficiently find and remove old scores from a heap
- Heap doesn't support removal by value

## Nuances to Remember

- Sorting approach is simpler and has same worst-case as lazy heap — prefer it unless addScore frequency >> top frequency
- `defaultdict(int)` gives 0 for missing keys — works naturally with `+= score`
- Lazy deletion heap copies the heap for each `top` call to avoid modifying the original
