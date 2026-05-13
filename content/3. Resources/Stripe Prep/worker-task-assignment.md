# Worker Task Assignment

**Date:** 2026-02-09
**Category:** Data Processing
**Parts Completed:** 4/4
**Language:** Python

## Problem Summary

Build a task assignment system for a support platform that distributes incoming tasks to available workers. Starts with simple load balancing and progressively adds specialty matching, account affinity, and worker availability handling.

## Solutions by Part

### Part 1: Load Balancing

**Approach:** Use a min-heap with `(workload, index, worker_name)` tuples. The tuple comparison handles both workload-based selection and tiebreaking by original list order automatically. Pop the min worker, assign the task, push back with updated workload.

```python
import heapq

def assign_tasks(workers, tasks):
    min_heap = []
    assignees = []
    for index, worker in enumerate(workers):
        heapq.heappush(min_heap, (0, index, worker))

    for task in tasks:
        taskId, duration = task["id"], task["duration"]
        workload, index, worker = heapq.heappop(min_heap)
        assignees.append({"taskId": taskId, "worker": worker})
        heapq.heappush(min_heap, (workload + duration, index, worker))
    return assignees
```

**Time Complexity:** O(T log W)

### Part 2: Specialty Matching

**Approach:** Switched from heap to dictionary-based tracking. Keep a `specialty_to_workers` map for lookup and a separate `worker_to_load` dict for workload. For each task, gather eligible workers across all required specialties (deduplicating with a set), then use `min()` with a key of `(workload, index)` to select.

```python
from collections import defaultdict

def assign_tasks(workers, tasks):
    speciality_to_workers = defaultdict(list)
    worker_to_load = defaultdict(int)
    worker_index = {}
    result = []

    for i, worker in enumerate(workers):
        name, speciality = worker["name"], worker["specialty"]
        speciality_to_workers[speciality].append(name)
        worker_index[name] = i

    for task in tasks:
        taskId, duration, requiredSpecialties = task["id"], task["duration"], task["requiredSpecialties"]
        possible_workers = set()
        for speciality in requiredSpecialties:
            curr_workers = speciality_to_workers[speciality]
            for worker in curr_workers:
                possible_workers.add(worker)
        min_worker = min(possible_workers, key=lambda w: (worker_to_load[w], worker_index[w]))
        worker_to_load[min_worker] += duration
        result.append({"taskId": taskId, "worker": min_worker})
    return result
```

**Time Complexity:** O(T × W)

### Part 3: Account Affinity

**Approach:** Added `worker_to_account` tracking (set per worker). For each task, partition eligible workers into those with account affinity and those without. Prefer affinity group; fall back to non-affinity group. Only update account history when a new worker-account pairing is created (in the else branch).

```python
from collections import defaultdict

def assign_tasks(workers, tasks):
    speciality_to_workers = defaultdict(list)
    worker_to_load = defaultdict(int)
    worker_to_account = defaultdict(set)
    worker_index = {}
    result = []

    for i, worker in enumerate(workers):
        name, speciality = worker["name"], worker["specialty"]
        speciality_to_workers[speciality].append(name)
        worker_index[name] = i

    for task in tasks:
        taskId, duration, requiredSpecialties, accountId = task["id"], task["duration"], task["requiredSpecialties"], task["accountId"]
        possible_workers = set()
        for speciality in requiredSpecialties:
            curr_workers = speciality_to_workers[speciality]
            for worker in curr_workers:
                possible_workers.add(worker)
        workers_with_account_affinity = [w for w in possible_workers if accountId in worker_to_account[w]]
        workers_with_no_account_affinity = [w for w in possible_workers if accountId not in worker_to_account[w]]
        if workers_with_account_affinity:
            min_worker = min(workers_with_account_affinity, key=lambda w: (worker_to_load[w], worker_index[w]))
        else:
            min_worker = min(workers_with_no_account_affinity, key=lambda w: (worker_to_load[w], worker_index[w]))
            worker_to_account[min_worker].add(accountId)
        worker_to_load[min_worker] += duration
        result.append({"taskId": taskId, "worker": min_worker})
    return result
```

### Part 4: Worker Availability

**Approach:** Pre-process offline events into a `worker_to_offline_idx` map. When building the eligible worker set, skip any worker whose offline index has been passed (`index > offline_idx`). Wrapped assignment logic in an `if possible_workers` guard for the case where all eligible workers are offline.

```python
from collections import defaultdict

def assign_tasks(workers, tasks, offlineEvents):
    speciality_to_workers = defaultdict(list)
    worker_to_load = defaultdict(int)
    worker_to_account = defaultdict(set)
    worker_index = {}
    result = []
    worker_to_offline_idx = {}
    for event in offlineEvents:
        worker, idx = event["worker"], event["afterTaskIndex"]
        worker_to_offline_idx[worker] = idx

    for i, worker in enumerate(workers):
        name, speciality = worker["name"], worker["specialty"]
        speciality_to_workers[speciality].append(name)
        worker_index[name] = i

    for index, task in enumerate(tasks):
        taskId, duration, requiredSpecialties, accountId = task["id"], task["duration"], task["requiredSpecialties"], task["accountId"]
        possible_workers = set()
        for speciality in requiredSpecialties:
            curr_workers = speciality_to_workers[speciality]
            for worker in curr_workers:
                if worker in worker_to_offline_idx and index > worker_to_offline_idx[worker]:
                    continue
                else:
                    possible_workers.add(worker)
        if possible_workers:
            workers_with_account_affinity = [w for w in possible_workers if accountId in worker_to_account[w]]
            workers_with_no_account_affinity = [w for w in possible_workers if accountId not in worker_to_account[w]]
            if workers_with_account_affinity:
                min_worker = min(workers_with_account_affinity, key=lambda w: (worker_to_load[w], worker_index[w]))
            else:
                min_worker = min(workers_with_no_account_affinity, key=lambda w: (worker_to_load[w], worker_index[w]))
                worker_to_account[min_worker].add(accountId)
            worker_to_load[min_worker] += duration
            result.append({"taskId": taskId, "worker": min_worker})
    return result
```

## Edge Cases

- **Empty tasks list** → returns empty result
- **Empty workers list** → exception on heappop (Part 1) or min() on empty sequence (Parts 2-4)
- **No matching specialty** → `possible_workers` is empty, task is skipped
- **All workers offline** → `possible_workers` is empty, task is skipped
- **Zero duration tasks** → don't affect workload ranking
- **min() on empty set** → would throw ValueError, guarded by `if possible_workers`

## Bugs & Issues

1. **Part 1:** Used `assertEqual` (unittest method) instead of `assert` — not a standalone function
2. **Part 2:** Initially appended lists into `possible_workers` instead of flattening — got list of lists
3. **Part 2:** `min()` was returning the min workload value, not the worker name — needed `key` function
4. **Part 2:** Variable shadowing — reused `workers` as loop variable, overwrote function parameter
5. **Part 3:** Used `defaultdict(tuple)` instead of `defaultdict(int)` for workload tracking
6. **Part 3:** Wrote `worker_to_load[worker]` instead of `worker_to_load[min_worker]` — wrong variable
7. **Part 4:** Off-by-one — used `>=` instead of `>` for offline check, which would skip the worker on the task they should still handle
8. **Part 4:** Typo `enuemrate` instead of `enumerate`

## Key Learnings

- **Start simple, evolve cleanly:** Part 1 used a heap (O(T log W)), but switching to linear scan for Part 2 made specialty filtering and later parts much cleaner to build on
- **Separate concerns in data structures:** Keep specialty mapping (static) separate from workload tracking (dynamic) to avoid stale data
- **`min()` with key tuples:** `min(candidates, key=lambda w: (workload[w], index[w]))` is a clean pattern for multi-criteria selection
- **Partition before select:** For affinity, split eligible workers into two groups and prefer one group — cleaner than a complex sorting key
- **Off-by-one with "after":** "afterTaskIndex: 2" means available AT index 2, unavailable AFTER — use `>` not `>=`

## Code Quality Notes

- Clean evolution from Part 1 → 4, each part layered on naturally
- Good use of `defaultdict` and `set` for lookups
- Could improve: extract worker selection into a helper function to reduce nesting in Part 4
- The `requiredSpecialties` key had a typo (`requiredSpecialities`) in early iterations — always match the spec exactly

## Q&A Highlights

- **Heap per specialty?** Considered having separate heaps per specialty, but workload staleness across heaps makes it impractical — simpler to use linear scan with a single workload dict
- **Account history update placement:** Only needed in the `else` (no affinity) branch since the `if` branch already has the account in history
- **afterTaskIndex semantics:** Clarified it refers to the global task list index, not the worker's personal task count
- **list vs set for account history:** Using a set gives O(1) lookup for affinity checks vs O(n) with a list
