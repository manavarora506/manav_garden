# Worker Task Assignment (Review Session)

**Date:** 2026-03-25
**Category:** Data Processing
**Parts Completed:** 4/4 (review)
**Language:** Python

## Problem Summary

Task scheduling system with load balancing, specialty matching, account affinity, and worker availability (offline events).

## Key Bugs Found on Review

### 1. Scanning all offline events every iteration — O(t × e)
```python
# Bug: loops ALL events for EVERY task
for event in offlineEvents:
    if task_index == afterTaskIndex:
        workers.remove(worker)

# Fix: pre-process into dict for O(1) lookup
offline_at = defaultdict(list)
for event in offlineEvents:
    offline_at[event["afterTaskIndex"]].append(event["worker"])

# In loop:
if task_index in offline_at:
    for worker in offline_at[task_index]:
        available.discard(worker)
```

### 2. list.remove() is O(n) — use set
```python
# Bug: O(n) per removal
workers.remove(worker)
# Fix: O(1) with set
available = set(worker["name"] for worker in self.workers)
available.discard(worker)
```

### 3. Implicit tiebreaker
```python
# Bug: relies on min() silently picking first item — fragile
min_worker = min(eligible, key=lambda x: self.worker_to_workload[x])
# Fix: explicit secondary sort key
worker_index = {w["name"]: i for i, w in enumerate(self.workers)}
min_worker = min(eligible, key=lambda x: (self.worker_to_workload[x], worker_index[x]))
```

## Final Clean Code

```python
from collections import defaultdict

class TaskScheduler:
    def __init__(self, workers, tasks, offlineEvents):
        self.workers = workers
        self.tasks = tasks
        self.worker_to_workload = {worker["name"]: 0 for worker in self.workers}
        self.worker_to_specialities = {worker["name"]: worker["specialty"] for worker in self.workers}
        self.worker_to_account_history = defaultdict(set)
        self.worker_index = {w["name"]: i for i, w in enumerate(self.workers)}
        self.available = set(worker["name"] for worker in self.workers)
        self.offline_at = defaultdict(list)
        for event in offlineEvents:
            self.offline_at[event["afterTaskIndex"]].append(event["worker"])

    def assign_tasks(self):
        result = []
        for task_index, task in enumerate(self.tasks):
            task_id = task["id"]
            duration = task["duration"]
            required_specialties = task["requiredSpecialties"]
            accountId = task["accountId"]
            eligible_workers = [
                w for w in self.available
                if self.worker_to_specialities[w] in required_specialties
            ]
            if not eligible_workers:
                continue
            with_affinity = [w for w in eligible_workers if accountId in self.worker_to_account_history[w]]
            candidates = with_affinity if with_affinity else eligible_workers
            min_worker = min(candidates, key=lambda x: (self.worker_to_workload[x], self.worker_index[x]))
            self.worker_to_workload[min_worker] += duration
            self.worker_to_account_history[min_worker].add(accountId)
            result.append({"taskId": task_id, "worker": min_worker})
            if task_index in self.offline_at:
                for worker in self.offline_at[task_index]:
                    self.available.discard(worker)
        return result
```

## Key Learnings

- Pre-process event lookups into dicts — avoid O(n) scans inside loops
- Use sets for membership checks and removal — O(1) vs O(n) for lists
- Make tiebreakers explicit in the sort key — don't rely on implementation details of min()
- Separate static lookups (specialty map) from dynamic state (workload, availability)
