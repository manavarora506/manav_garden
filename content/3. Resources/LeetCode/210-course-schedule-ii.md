# 210. Course Schedule II

**Date:** 2026-01-31
**Difficulty:** Medium
**Topics:** Topological Sort, BFS, Graph, Kahn's Algorithm
**Link:** https://leetcode.com/problems/course-schedule-ii/

## Final Solution

```python
class Solution:
    def findOrder(self, numCourses: int, prerequisites: List[List[int]]) -> List[int]:
        reverse_adj_list = defaultdict(list)
        in_edge = [0 for i in range(numCourses)]
        for course, prereq in prerequisites:
            reverse_adj_list[prereq].append(course)
            in_edge[course] += 1
        queue = deque()
        result = []
        # add all nodes with indegree 0
        for c in range(numCourses):
            if in_edge[c] == 0:
                queue.append(c)
        while queue:
            course = queue.popleft()
            result.append(course)
            dependencies = reverse_adj_list[course]
            for dependent in dependencies:
                in_edge[dependent] -= 1
                if in_edge[dependent] == 0:
                    queue.append(dependent)
        if len(result) == numCourses:
            return result
        else:
            return []
```

## Initial Issues

- **Key challenge:** Recognizing that `len(result) == numCourses` is how you detect cycles
  - If there's a cycle, some nodes never reach in-degree 0 and won't be processed

## Key Learnings

- **Kahn's Algorithm for Topological Sort:**
  1. Count incoming edges (in-degree) for all nodes
  2. Start with nodes that have no prerequisites (in-degree 0)
  3. "Take" a course → decrement in-degree of all dependents
  4. When a dependent hits in-degree 0, it's ready to take
  5. If you can't take all courses → cycle exists

- **Edge direction matters:**
  - Input: `[course, prereq]` means prereq → course
  - Adjacency list: `prereq: [courses that depend on it]`
  - In-degree tracks: how many prerequisites each course has

## Nuances to Remember

- **Cycle detection:** `len(result) != numCourses` means a cycle exists (some nodes stuck with in-degree > 0)
- **Variable shadowing:** Avoid reusing loop variable names from outer scope (e.g., use `dependent` not `course` in inner loop)
- **defaultdict(list):** Useful for adjacency lists - no need to check if key exists

## Mental Model

```
Prerequisites: [[1,0], [2,1]]  means  0 → 1 → 2

in_degree: [0, 1, 1]  (course 0 has no prereqs)
Start with course 0 (in-degree 0)
Take 0 → decrement 1's in-degree → 1 becomes 0 → add to queue
Take 1 → decrement 2's in-degree → 2 becomes 0 → add to queue
Take 2
Result: [0, 1, 2]
```
