# 269. Alien Dictionary

**Date:** 2026-01-31
**Difficulty:** Hard
**Topics:** Topological Sort, BFS, Graph, Kahn's Algorithm
**Link:** https://leetcode.com/problems/alien-dictionary/

## Final Solution

```python
class Solution:
    def alienOrder(self, words: List[str]) -> str:
        edges = []
        nodes = set()
        result = []
        
        # Collect all unique characters
        for word in words:
            for char in word:
                nodes.add(char)
        
        # Extract ordering relationships from adjacent words
        for i in range(len(words) - 1):
            currWord = words[i]
            nextWord = words[i+1]
            minLen = min(len(currWord), len(nextWord))
            foundDiff = False
            for j in range(minLen):
                currLetter = currWord[j]
                nextLetter = nextWord[j]
                if currLetter != nextLetter:
                    edges.append([currLetter, nextLetter])
                    foundDiff = True
                    break
            if not foundDiff and len(currWord) > len(nextWord):
                return ""
        
        # Build graph and run topological sort
        queue = deque()
        adj_list = {node: [] for node in nodes}
        in_edges = {node: 0 for node in nodes}

        for letter, nextLetter in edges:
            adj_list[letter].append(nextLetter)
            in_edges[nextLetter] += 1
        
        for letter in in_edges:
            if in_edges[letter] == 0:
                queue.append(letter)
        
        while queue:
            letter = queue.popleft()
            result.append(letter)
            dependencies = adj_list[letter]
            for dep in dependencies:
                in_edges[dep] -= 1
                if in_edges[dep] == 0:
                    queue.append(dep)
        
        return "".join(result) if len(result) == len(nodes) else ""
```

## Initial Issues

1. **Edge extraction loop bug:** Used `for j in range(currWord)` instead of `for j in range(len(currWord))`

2. **Missing break:** Didn't break after finding first difference - only the first differing character tells you ordering

3. **Invalid prefix case:** Didn't handle case where `currWord` is longer and `nextWord` is a prefix (e.g., ["abc", "ab"] is invalid). Need to check this only when no difference was found (`for...else` or flag)

4. **Node collection bug:** Initially tried to collect nodes during edge comparison loop, but this misses characters that appear after the first difference

5. **Adjacency list direction wrong:** Built `adj_list[nextLetter].append(letter)` (backwards) instead of `adj_list[letter].append(nextLetter)` (forward). When processing a node, need to decrement in_degree of nodes that come AFTER it, not before.

## Key Learnings

- **Core insight:** Comparing adjacent words reveals character ordering. First differing character tells you `currWord[j]` comes before `nextWord[j]`

- **Two-phase approach:**
  1. Extract edges from word comparisons
  2. Run topological sort on the character graph

- **Invalid input cases:**
  - Prefix case: ["abc", "ab"] - longer word can't come before its prefix
  - Cycle: detected by `len(result) != len(nodes)`

- **Adjacency list direction matters:** When node A → B (A before B):
  - Store B in A's adjacency list
  - When A is processed, decrement B's in_degree

## Nuances to Remember

- Collect ALL unique characters first, not during edge extraction
- Only first differing character between adjacent words gives ordering info
- `for...else` pattern: else runs only if loop completes without break
- Characters with no ordering constraints still need to appear in output (in_degree 0 from start)

## Edge Extraction Pattern

```python
for j in range(min(len(word1), len(word2))):
    if word1[j] != word2[j]:
        # word1[j] comes before word2[j]
        edges.append([word1[j], word2[j]])
        break
else:
    # No difference found - check prefix case
    if len(word1) > len(word2):
        return ""  # Invalid
```
