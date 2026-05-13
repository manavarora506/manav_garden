# 207. Course Schedule

**Date:** 2026-01-26
**Difficulty:** Medium
**Topics:** Topological Sort, Kahn's Algorithm, BFS, Graph, Cycle Detection
**Link:** https://leetcode.com/problems/course-schedule/

## Final Solution

```python
def canFinish(self, numCourses: int, prerequisites: List[List[int]]) -> bool:
    queue = deque()
    reverse_adj_list = {i: [] for i in range(numCourses)}
    in_degree = {i: 0 for i in range(numCourses)}
    for course, prereq in prerequisites:
        in_degree[course] += 1
        reverse_adj_list[prereq].append(course)
    
    for course in range(numCourses):
        if in_degree[course] == 0:
            queue.append(course)
    
    processed = 0
    while queue:
        currCourse = queue.popleft()
        processed += 1
        for course in reverse_adj_list[currCourse]:
            in_degree[course] -= 1
            if in_degree[course] == 0:
                queue.append(course)
    
    return processed == numCourses
```

## Initial Issues

1. **Started BFS from only course 0** - Missed that the graph could be disconnected; need to start from ALL courses with no prerequisites.

2. **Wrong cycle detection logic** - Initially thought "visited = cycle", but visiting a node twice from different paths isn't a cycle. A cycle means nodes are stuck waiting on each other.

3. **Didn't understand the need for reverse adjacency list** - Need to know "which courses depend on this one" to decrement their in_degree when a course is completed.

4. **Variable name typo** - Iterated with `prereqs` but checked `prereq`.

5. **Confused about visited set** - Kahn's doesn't need a visited set; in_degree naturally prevents re-processing since it only decrements, never increments.

## Key Learnings

### Kahn's Algorithm Mental Model

**One sentence:** Keep taking courses that have no remaining prerequisites, until you can't take any more.

**The pieces:**
- `in_degree[course]` = "how many prereqs do I still need?"
- `reverse_adj_list[course]` = "who's waiting on me?"
- Queue = courses ready to take (in_degree == 0)

**The loop:**
1. Take a ready course
2. Tell everyone waiting on it: "one less prereq to worry about"
3. If that makes someone else ready, add them to the queue

### Why This Detects Cycles

If there's a cycle (A requires B, B requires A):
- Both start with in_degree = 1
- Neither ever reaches in_degree = 0
- Neither gets added to queue
- Neither gets processed
- `processed < numCourses` → cycle exists

### No Visited Set Needed

A course only enters the queue when `in_degree == 0`. You only decrement, never increment. So a course can only reach 0 once → can only be added once.

## Nuances to Remember

- **in_degree** = incoming edges = number of prerequisites
- **Don't need level tracking** - Unlike shortest path BFS, we just need to process all nodes, not track distance
- **Count processed nodes** - Compare to numCourses to detect if cycle trapped some nodes
- **Reverse adjacency list** - Essential for knowing who to "unlock" when completing a course

## Q&A Highlights

**Q: Isn't in_degree the same as len(adj_list[course])?**
A: Initially yes, but you need a separate counter to decrement during the algorithm.

**Q: When should courses be marked as visited?**
A: You don't need visited. In_degree naturally prevents re-processing - a course only gets added when in_degree hits 0, which can only happen once.

**Q: What's the difference between adj_list and reverse_adj_list?**
A: adj_list[course] = prerequisites for this course. reverse_adj_list[prereq] = courses that depend on this prereq. Kahn's only needs the reverse list.
