# 323. Number of Connected Components in an Undirected Graph

**Date:** 2026-02-02
**Difficulty:** Medium
**Topics:** DFS, BFS, Union-Find, Graph
**Link:** https://leetcode.com/problems/number-of-connected-components-in-an-undirected-graph/

## DFS Solution

```python
def countComponents(self, n: int, edges: List[List[int]]) -> int:
    adj_list = defaultdict(list)
    for source, dest in edges:
        adj_list[source].append(dest)
        adj_list[dest].append(source)
    
    def dfs(source):
        if source in visited:
            return
        visited.add(source)
        for edge in adj_list[source]:
            if edge not in visited:
                dfs(edge)

    numComponents = 0
    visited = set()
    for i in range(n):
        if i not in visited:
            numComponents += 1
            dfs(i)

    return numComponents
```

## BFS Solution

```python
def countComponents(self, n: int, edges: List[List[int]]) -> int:
    adj_list = defaultdict(list)
    for source, dest in edges:
        adj_list[source].append(dest)
        adj_list[dest].append(source)
    
    def bfs(start):
        my_queue = deque([start])
        visited.add(start)  # Mark when adding!
        while my_queue:
            node = my_queue.popleft()
            for edge in adj_list[node]:
                if edge not in visited:
                    visited.add(edge)  # Mark when adding!
                    my_queue.append(edge)

    numComponents = 0
    visited = set()
    for i in range(n):
        if i not in visited:
            numComponents += 1
            bfs(i)

    return numComponents
```

## Initial Issues

**BFS bug:** Marked visited when popping instead of when adding to queue. This causes duplicates in the queue.

## Key Learnings

### Counting Connected Components Pattern

1. Build adjacency list (undirected = add both directions)
2. For each node 0 to n-1:
   - If not visited, increment component count
   - Run DFS/BFS from that node to mark all reachable nodes
3. Return count

### BFS: When to Mark Visited

**Wrong - mark when popping:**
```python
node = queue.popleft()
visited.add(node)  # Too late!
```

**Right - mark when adding:**
```python
if edge not in visited:
    visited.add(edge)  # Immediately
    queue.append(edge)
```

**Why it matters:**
If A and B both connect to C, and you process A first:
- Wrong: Add C from A, then add C again from B (C not marked yet)
- Right: Add C from A and mark it, skip C from B (C already marked)

## Nuances to Remember

| Traversal | Mark visited when... |
|-----------|---------------------|
| DFS | Entering the node (start of function) |
| BFS | Adding to queue (before pushing) |

- Undirected graph: add both `adj[a].append(b)` and `adj[b].append(a)`
- Nodes are 0 to n-1, so iterate `range(n)` to catch disconnected nodes
