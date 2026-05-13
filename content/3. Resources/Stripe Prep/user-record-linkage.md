# User Record Linkage

**Date:** 2026-02-09
**Category:** Data Processing / API Design
**Parts Completed:** 3/3
**Language:** Python

## Problem Summary

Build a user deduplication system that links records based on weighted field similarity. Given user records with name, email, and company fields, each with a similarity weight, find all records that match a target user above a threshold. Progressive parts: direct matches, one-hop indirect matches, and unlimited-hop connected component discovery.

## Solutions by Part

### Part 1: Direct Matches

**Approach:** Compare the target record against all other records. For each field, if values match exactly, add that field's weight to the similarity score. If total score >= threshold, it's a match.

### Part 2: One-Hop Indirect Matches

**Approach:** BFS with depth limit of 2. Seed queue with target record, explore direct matches (depth 1), then explore their matches (depth 2). Use visited set to avoid revisiting records.

### Part 3: Unlimited Hops (Full Connected Component)

**Approach:** Same BFS but remove depth limit. Let BFS explore until queue is empty. Visited set prevents cycles. Finds the entire connected component reachable from the target.

### Final Solution (All Parts)

```python
from collections import deque

def find_linked(rows: list, weights: dict, threshold: int, target_user_id: int) -> list:
    def _calc_similarity(row_name, target_name, row_email, target_email, row_company, target_company):
        similarity = 0
        name_weight, email_weight, company_weight = weights["name"], weights["email"], weights["company"]
        if row_name == target_name:
            similarity += name_weight
        if row_email == target_email:
            similarity += email_weight
        if row_company == target_company:
            similarity += company_weight
        return similarity

    matches = []
    queue = deque()
    target_row = next(row for row in rows if row["id"] == target_user_id)
    visited = set()
    visited.add(target_user_id)
    queue.append(target_row)

    while queue:
        currRow = queue.popleft()
        row_id = currRow["id"]
        row_name, row_email, row_company = currRow["name"], currRow["email"], currRow["company"]
        for row in rows:
            if row_id != row["id"]:
                curr_id = row["id"]
                curr_name, curr_email, curr_company = row["name"], row["email"], row["company"]
                if curr_id not in visited:
                    visited.add(curr_id)
                    similarity = _calc_similarity(row_name, curr_name, row_email, curr_email, row_company, curr_company)
                    if similarity >= threshold:
                        queue.append(row)
                        matches.append(curr_id)

    return matches
```

**For Part 2 (depth-limited):** Add depth tracking to queue entries and check `if depth < 2` before enqueuing.

## Edge Cases

- **Target user not in rows** → `next()` would raise StopIteration; could add default
- **No matches at all** → returns empty list
- **All records match each other** → returns all IDs except target
- **Self-comparison** → prevented by `row_id != row["id"]` check and adding target to visited
- **Threshold of 0** → everything matches
- **Threshold of 1.0** → all fields must match exactly

## Bugs & Issues

1. **`rows[target_user_id]` assumed IDs are array indices** — IDs don't necessarily match array positions; used `next()` to find by ID
2. **`_calc_similarity` parameter named `target_row` but compared `target_name`** — parameter name didn't match usage
3. **`queue` never initialized** — forgot `queue = deque()` before using it
4. **`target_user_id` not in visited set initially** — could match target against itself
5. **`adj_list` referenced without `self.`** — from earlier currency converter habit, but not applicable here
6. **`row_by_id` dict built but never used** — dead code, can remove
7. **Typo: `similariy` instead of `similarity`** — variable name mismatch

## Key Learnings

- **BFS depth limit pattern** — add depth to queue tuple, check before enqueuing. Same pattern as currency converter
- **Part 1 → Part 2 → Part 3 is just removing constraints** — direct (depth 1) → one-hop (depth 2) → unlimited (no depth check)
- **`next()` for finding a record by ID** — cleaner than loop-and-break, stops at first match
- **Generators are lazy** — `(x for x in list if condition)` doesn't evaluate until `next()` or iteration
- **Seed BFS with the target** — avoids duplicate code for initial matches vs hop matches
- **Visited set prevents cycles** — no need for per-path visited since we want the full component (unlike currency converter where we wanted all paths)

## Code Quality Notes

- `_calc_similarity` could be generalized to loop over `weights.keys()` instead of hardcoding field names
- `row_by_id` was built but unused — remove dead code
- Could extract field unpacking into a helper to reduce repetition
- Visited set shared across all exploration is correct here (unlike currency converter) because we want the component, not all paths

## Q&A Highlights

- **Why shared visited set here but per-path in currency converter?** Currency converter needs all paths to find the best rate. Here we just need reachability — once a record is found linked, it stays linked regardless of path
- **`next()` vs list comprehension** — `next()` stops at first match (O(1) best case), list comprehension scans everything then indexes
- **How to limit to one hop?** Add depth to queue entries: `queue.append((row, depth + 1))` and check `if depth < 2`


## Refactored Solution

Cleaner version with generalized similarity calculation:

```python
from collections import deque

def find_linked(rows: list, weights: dict, threshold: int, target_user_id: int) -> list:
    def _calc_similarity(row, target):
        return sum(weights[field] for field in weights if row[field] == target[field])

    matches = []
    queue = deque()
    target_row = next(row for row in rows if row["id"] == target_user_id)
    visited = set()
    visited.add(target_user_id)
    queue.append(target_row)

    while queue:
        currRow = queue.popleft()
        row_id = currRow["id"]
        for row in rows:
            if row_id != row["id"]:
                curr_id = row["id"]
                if curr_id not in visited:
                    visited.add(curr_id)
                    similarity = _calc_similarity(row, currRow)
                    if similarity >= threshold:
                        queue.append(row)
                        matches.append(curr_id)

    return matches
```

Key improvements over initial version:
- `_calc_similarity` takes full row dicts and loops over `weights` keys — no hardcoded field names
- Removed unused `row_by_id`, `target_name/email/company` variables
- Automatically adapts if new weighted fields are added
