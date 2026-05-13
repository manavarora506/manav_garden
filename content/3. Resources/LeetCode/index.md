# LeetCode Progress

Tracking problems practiced with detailed notes on bugs, learnings, and nuances.

## Problems

- [[752-open-lock]] - BFS level tracking, visited timing, Python modulo with negatives
- [[279-perfect-squares]] - Graph modeling insight, BFS for "minimum operations" problems, continue vs break
- [[207-course-schedule]] - Kahn's algorithm, topological sort, cycle detection via in-degree
- [[1971-find-if-path-exists-in-graph]] - Basic BFS path existence, undirected graph traversal

- [[797-all-paths-from-source-to-target]] - BFS all paths in DAG; path reference copying gotcha

- [[116-populating-next-right-pointers-in-each-node]] - BFS level traversal; O(1) space uses next pointers as linked list (revisit)

- [[1091-shortest-path-in-binary-matrix]] - BFS shortest path; 8-direction grid; mutate grid for O(1) space

- [[429-n-ary-tree-level-order-traversal]] - BFS level order; n-ary just loops over children list

- [[210-course-schedule-ii]] - Kahn's topological sort; cycle detection via result length check

- [[1136-parallel-courses]] - Topological sort + level counting; min semesters = BFS depth

- [[269-alien-dictionary]] - Hard; extract ordering from adjacent words + topological sort; adjacency direction matters

- [[310-minimum-height-trees]] - ⚠️ REVISIT; peel leaves to find tree center; track remaining nodes not queue size

- [[1971-find-if-path-exists-in-graph]] - DFS existence check, when to backtrack vs shared visited set

- [[797-all-paths-from-source-to-target]] - DFS backtracking, DAG vs general graph visited set requirements

- [[1059-all-paths-from-source-lead-to-destination]] - DFS with visited/verified sets, cycle detection in directed graphs, topo sort alternative

- [[133-clone-graph]] - Graph cloning with dict mapping, add to dict before recursing to handle cycles

- [[215-kth-largest-element-in-an-array]] - Min heap of size k, kth largest at root, O(n log k)

- [[1046-last-stone-weight]] - Max heap via negation, heapify returns None, simulation

- [[973-k-closest-points-to-origin]] - Max heap for k smallest, tuple order matters, no sqrt needed

- [[355-design-twitter]] - ⚠️ REDO - Merge k sorted lists with heap, many implementation bugs

- [[621-task-scheduler]] - ⚠️ REDO - Heap + queue simulation, or math formula, cooldown scheduling

- [[323-number-of-connected-components-in-an-undirected-graph]] - DFS/BFS traversal, mark visited when adding to queue

- [[94-binary-tree-inorder-traversal]] - Recursive vs iterative, nonlocal usage, stack simulates recursion

- [[144-binary-tree-preorder-traversal]] - Push right then left for correct order, simpler than inorder iterative

- [[285-inorder-successor-in-bst]] - BST search from root, track successor when going left, O(H) iterative

- [[285-inorder-predecessor-in-bst]] - Mirror of successor; track predecessor when going right, O(H) iterative

- [[173-binary-search-tree-iterator]] - Controlled inorder with stack; amortized O(1) next(), O(H) space

- [[700-search-in-a-binary-search-tree]] - Basic BST search; iterative O(1) space vs recursive O(H) space

- [[701-insert-into-a-binary-search-tree]] - Insert as leaf; recursive is cleaner, iterative needs parent tracking

- [[206-reverse-linked-list]] - Recursive reversal; head.next.next pointer manipulation, unwinding call stack

- [[118-pascals-triangle]] - Iterative and recursive row building; bucket brigade return pattern

- [[119-pascals-triangle-ii]] - Recursive single-row return; O(rowIndex) space optimization; minimal base cases

- [[139-word-break]] - Recursive consume-from-front; memoize failed suffixes; O(n*m*k) with cache

- [[394-decode-string]] - Stack with `[` boundary marker; prepend when popping to preserve decoded string order

- [[208-implement-trie]] - Trie insert/search/startsWith/delete; count-based deletion; when trie vs hash set

- [[119-pascals-triangle-ii]] - Recursive single-row optimization; only rowIndex==0 base case needed
- [[139-word-break]] - Consume from front with startsWith; memoize failed suffixes; O(n*m*k)

- [[custom-unique-elements-across-k-lists]] - Set of list IDs per num; MapReduce extension with hash partitioning

- [[1244-design-a-leaderboard]] - Sorting vs lazy deletion heap; score accumulation; seen set for dedup

- [[78-subsets]] - Include/exclude backtracking; binary decision tree; 2^n subsets

- [[39-combination-sum]] - Include/exclude with reuse; "how many of each" mental model; prune on sum > target

- [[46-permutations]] - Loop all + seen set; permutations vs subsets pattern; backtrack inside loop

- [[90-subsets-ii]] - Sort + skip duplicates in exclude branch only; pruning prevents redundant paths

- [[22-generate-parentheses]] - Backtrack with open/close counts; 2^(2n) tree pruned to Catalan number

- [[79-word-search]] - Grid backtracking; mark/restore outside loop; O(m*n*4^L) time

- [[backtracking-complexity-reference]] - Framework: choices^depth; cheat sheet for all backtracking problems

- [[17-letter-combinations-of-a-phone-number]] - Digit-to-letter backtracking; O(n * 4^n) time; auxiliary vs total space

- [[74-search-a-2d-matrix]] - Binary search on virtual flat array; mid//cols and mid%cols conversion

- [[153-find-minimum-in-rotated-sorted-array]] - Compare mid vs end; start < end to narrow to one; loop condition choice
