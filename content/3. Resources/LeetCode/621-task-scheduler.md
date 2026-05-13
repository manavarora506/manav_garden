# 621. Task Scheduler

**Date:** 2026-02-02
**Difficulty:** Medium
**Topics:** Heap, Greedy, Queue
**Link:** https://leetcode.com/problems/task-scheduler/

> ⚠️ **TODO:** Practice this problem again - needed significant help with implementation.

## Final Solution (Heap + Queue Approach)

```python
def leastInterval(self, tasks: List[str], n: int) -> int:
    from collections import Counter, deque
    import heapq
    
    freq = Counter(tasks)
    available = [-cnt for cnt in freq.values()]  # Max heap (negated)
    heapq.heapify(available)
    not_available = deque()  # (count, available_time)
    
    time = 0
    while available or not_available:
        time += 1
        
        # Move tasks that are now available back to heap
        while not_available and not_available[0][1] <= time:
            cnt, _ = not_available.popleft()
            heapq.heappush(available, cnt)
        
        if available:
            cnt = heapq.heappop(available)  # cnt is negative
            cnt += 1  # Decrement (less negative)
            if cnt < 0:  # Still has remaining tasks
                not_available.append((cnt, time + n + 1))
        # else: idle time
    
    return time
```

## Alternative: Math Formula Approach

```python
def leastInterval(self, tasks: List[str], n: int) -> int:
    freq = Counter(tasks)
    max_freq = max(freq.values())
    count_max = sum(1 for f in freq.values() if f == max_freq)
    
    formula = (max_freq - 1) * (n + 1) + count_max
    return max(len(tasks), formula)
```

## Initial Issues

1. **Missing heap argument**: `heapq.heappop()` instead of `heapq.heappop(available)`

2. **Wrong count decrement**: `cnt -= 1` when cnt is negative. Should be `cnt += 1`

3. **Wrong method name**: `appendRight` instead of `append`

4. **Backwards condition**: `if currTime >= time` instead of `if time >= currTime`

5. **Modifying while iterating**: Iterating over deque while popping from it

6. **Missing idle time handling**: `while available` instead of `while available or not_available`

7. **Time not incrementing**: Forgot `time += 1`

8. **Mixed data types**: Pushed tuples to heap but tried to treat popped value as int

## Key Learnings

### Two Approaches

**1. Heap + Queue (Simulation):**
- Max heap: tasks available to schedule (by frequency)
- Queue: tasks cooling down with their available_time
- Each step: move available tasks from queue to heap, pop from heap, schedule

**2. Math Formula:**
- `(max_freq - 1) * (n + 1) + count_max`
- Answer: `max(total_tasks, formula)`

### Why Prioritize High Frequency Tasks?

Tasks with highest frequency are most likely to cause idle time. Schedule them first to spread them out.

### Cycle-Based Intuition

With cooldown `n`, you can schedule at most `n+1` different tasks before the first one is available again. This guarantees the cooldown is satisfied.

```
n=2, cycle size = 3
A B C | A B C | A B  (each A is 3 slots apart)
```

## Nuances to Remember

- Heap stores **negative counts** for max heap behavior
- Queue stores **(count, available_time)** tuples
- Check `cnt < 0` not `cnt > 0` (counts are negative)
- Loop condition: `while heap or queue` to handle idle time
- Move from queue to heap when `time >= available_time`

## Q&A Highlights

**Q:** How is cooldown satisfied in the cycle approach?
**A:** Each task appears at most once per cycle of (n+1) slots. So there's always at least n slots between occurrences of the same task.

**Q:** When do you need idle time?
**A:** When you don't have enough different tasks to fill the gaps. Formula result > total tasks means idle time needed.
