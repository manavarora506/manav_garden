# 1136. Parallel Courses

**Date:** 2026-01-31
**Difficulty:** Medium
**Topics:** Topological Sort, BFS, Graph, Kahn's Algorithm
**Link:** https://leetcode.com/problems/parallel-courses/

## Final Solution

```python
class Solution:
    def minimumSemesters(self, n: int, relations: List[List[int]]) -> int:
        in_degree = {i: 0 for i in range(1, n+1)}
        reverse_adj_list = defaultdict(list)
        queue = deque()
        for prevCourse, nextCourse in relations:
            reverse_adj_list[prevCourse].append(nextCourse)
            in_degree[nextCourse] += 1
        for course in in_degree:
            if in_degree[course] == 0:
                queue.append(course)
        minSemesters = 0
        coursesVisited = 0
        while queue:
            size = len(queue)
            for i in range(size):
                course = queue.popleft()
                coursesVisited += 1
                dependencies = reverse_adj_list[course]
                for dep in dependencies:
                    in_degree[dep] -= 1
                    if in_degree[dep] == 0:
                        queue.append(dep)
            minSemesters += 1
        return minSemesters if coursesVisited == n else -1
```

## Initial Issues

- None significant - solved cleanly building on Course Schedule II pattern

## Key Learnings

- **One BFS level = one semester:** All courses with in-degree 0 at the start of a level can be taken in parallel
- **Minimum semesters = number of BFS levels** = longest path in DAG
- Same Kahn's algorithm as Course Schedule II, just tracking levels instead of order
- Courses labeled 1 to n (not 0 to n-1) - use `range(1, n+1)`

## Nuances to Remember

- `size = len(queue)` at start of each level to process all "ready" courses in one semester
- Cycle detection: `coursesVisited == n` check at the end
- Early exit `if len(queue) == 0: return -1` is valid but redundant (final check handles it)

## Pattern Comparison

| Problem | What you track |
|---------|----------------|
| Course Schedule II | Order of courses taken |
| Parallel Courses | Number of levels (semesters) |

Both use Kahn's algorithm (BFS topological sort), just returning different things.

## Connection to BFS Patterns

This combines:
- **Level-order traversal** (counting levels with `size = len(queue)`)
- **Topological sort** (processing by in-degree)

"Minimum semesters" = "How many levels deep is this DAG?"
