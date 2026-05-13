# 208. Implement Trie (Prefix Tree)

**Date:** 2026-03-17
**Difficulty:** Medium
**Topics:** Trie, Hash Map, Design
**Link:** https://leetcode.com/problems/implement-trie-prefix-tree/

## Final Solution

```python
class TrieNode:
    def __init__(self, value=None, count=0):
        self.value = value
        self.children = {}
        self.count = count
        self.isEndOfWord = False

class Trie:

    def __init__(self):
        self.root = TrieNode()

    def insert(self, word: str) -> None:
        curr = self.root
        for c in word:
            if c not in curr.children:
                cTrieNode = TrieNode(c, 1)
                curr.children[c] = cTrieNode
            else:
                curr.children[c].count += 1
            curr = curr.children[c]
        curr.isEndOfWord = True

    def search(self, word: str) -> bool:
        curr = self.root
        for c in word:
            if c not in curr.children:
                return False
            curr = curr.children[c]
        return curr.isEndOfWord

    def startsWith(self, prefix: str) -> bool:
        curr = self.root
        for c in prefix:
            if c not in curr.children:
                return False
            curr = curr.children[c]
        return True

    def helper(self, word, index, node):
        if index == len(word):
            node.isEndOfWord = False
            return node
        letter = word[index]
        curr = node.children[letter]
        self.helper(word, index + 1, curr)
        node.children[letter].count -= 1
        if node.children[letter].count == 0:
            del node.children[letter]
        return node

    def deleteWord(self, word):
        if self.search(word):
            self.helper(word, 0, self.root)
```

## Initial Issues

- Accessed `node[c]` instead of `node.children[c]` — TrieNode is not a dict
- Used undefined variable `c` instead of `letter` in recursive call
- Checked `letter.count` (string has no count) instead of `node.children[letter].count`
- Forgot `self.` before `helper()` method call
- Incremented wrong node's count in insert (`curr.count` instead of `curr.children[c].count`)

## Key Learnings

### What is a Trie
- Tree where each path from root represents a prefix of a stored string
- Each node's `children` is a **dict** mapping characters to child TrieNodes
- One node can have multiple children (up to 26 for lowercase English)
- `isEndOfWord` flag distinguishes complete words from prefixes (e.g., "cat" vs "cats")

### When to use a Trie vs Hash Set
- **Use trie:** prefix queries, autocomplete, word-by-word matching (Word Search II)
- **Use hash set:** simple exact word lookups — O(1) average and simpler

### Delete with count approach
- `count` tracks how many words pass through each node
- Insert: increment count at each node along the path
- Delete: recurse to end, set `isEndOfWord = False`, then decrement count on the way back up. If count hits 0, delete the node
- Same "bucket brigade" recursion pattern — recurse down, clean up on the way back

### Delete call stack trace (deleting "cat" from trie with "cat" and "car")
```
root → 'c'(2) → 'a'(2) → 't'(1)✓
                         → 'r'(1)✓

helper("cat", 3, node_t) → set isEndOfWord = False, return
helper("cat", 2, node_a) → t.count=0 → delete 't' 🗑️
helper("cat", 1, node_c) → a.count=1 → keep ✓
helper("cat", 0, root)   → c.count=1 → keep ✓

Result: root → 'c'(1) → 'a'(1) → 'r'(1)✓
```

## Nuances to Remember

- `search` and `startsWith` are nearly identical — only difference is `search` checks `isEndOfWord`, `startsWith` just returns `True`
- Time: O(word length) for insert, search, startsWith
- Space: O(total characters across all words) worst case
- Common trie problems: 208, 211 (wildcard search), 212 (Word Search II), 648 (Replace Words)
