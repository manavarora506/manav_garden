# Currency Exchange Rate Converter

**Date:** 2026-02-09
**Category:** API Design / Data Processing
**Parts Completed:** 4/4
**Language:** Python

## Problem Summary

Build a currency exchange system that converts amounts between currencies using known exchange rates. Parse rates from a string format, support forward and reverse lookups, single intermediate conversions, optimal rate finding across multiple paths, and unlimited conversion chains via graph traversal.

## Solutions by Part

### Part 1: Direct Conversion with Reverse Lookup

**Approach:** Parse the rate string, store both forward and reverse rates in a dictionary with keys like `"USD:EUR"`. Reverse rate = `1/rate`. O(1) lookup via string key.

### Part 2: Single Intermediate Conversion

**Approach:** Build an adjacency list alongside the rate dictionary. If no direct rate, check neighbors of the source currency for a path through one intermediate. Can also use BFS with depth limit of 2.

### Part 3: Optimal Exchange Rate

**Approach:** BFS exploring all paths (not just first found). Track `maxExchange` across all paths that reach the destination. Each path gets its own visited set to allow independent exploration.

### Part 4: Unlimited Conversion Chain

**Approach:** Same BFS as Part 3 with no depth limit. Each queue entry carries `(currency, accumulated_rate, visited_set)`. Creates a copy of visited set per path to avoid mutation bugs.

### Combined Final Solution

```python
from collections import defaultdict, deque

class CurrencyConverter():
    def __init__(self, rates: str):
        self.rates = rates
        self.rate_to_conversion = {}
        self.adj_list = defaultdict(list)
        self.build_dict()

    def build_dict(self) -> None:
        rates_list = self.rates.split(",")
        for rate in rates_list:
            rate_list = rate.split(":")
            countryA, countryB, exchange = rate_list[0], rate_list[1], float(rate_list[2])
            key = f"{countryA}:{countryB}"
            reverse_key = f"{countryB}:{countryA}"
            self.adj_list[countryA].append(countryB)
            self.adj_list[countryB].append(countryA)
            self.rate_to_conversion[key] = exchange
            self.rate_to_conversion[reverse_key] = float(1 / exchange)

    def getRate(self, countryA: str, countryB: str) -> float:
        key = f"{countryA}:{countryB}"
        if key in self.rate_to_conversion:
            return self.rate_to_conversion[key]
        else:
            queue = deque()
            visited = set()
            visited.add(countryA)
            queue.append((countryA, 1.0, visited))
            maxExchange = -1
            while queue:
                currCountry, currExchange, currPath = queue.popleft()
                if currCountry == countryB:
                    maxExchange = max(maxExchange, currExchange)
                    continue
                for edge in self.adj_list[currCountry]:
                    if edge not in currPath:
                        newPath = currPath.copy()
                        newPath.add(edge)
                        key = f"{currCountry}:{edge}"
                        edgeExchange = self.rate_to_conversion[key]
                        new_exchange = currExchange * edgeExchange
                        queue.append((edge, new_exchange, newPath))
        return maxExchange
```

## Edge Cases

- **Same currency conversion** → should return 1.0 (not currently handled, worth adding)
- **Currency not in the system** → returns -1 (no paths found)
- **Empty rate string** → adj_list and rate_to_conversion stay empty
- **Cycles in currency graph** → handled by visited set per path
- **Single currency pair** → direct lookup, BFS not needed

## Bugs & Issues

1. **`build_dict` took unnecessary parameters** — used `self.` attributes inside anyway, params were redundant
2. **`adj_list` initialized with `range(rates_list)`** — `range()` takes int not list; should use `defaultdict(list)`
3. **Missing reverse edges in adj_list** — initially only added `countryA → countryB`, not `countryB → countryA`
4. **`adj_list` referenced without `self.`** — wrote `adj_list[currCountry]` instead of `self.adj_list[currCountry]`
5. **Unnecessary inner `for i in range(size)` loop** inside BFS — BFS `while queue` + `popleft` already processes one node at a time
6. **Mutating shared visited set** — `currPath.add(edge)` mutated the same set across all paths; fixed with `currPath.copy()` + `.add()`
7. **Missing `continue` after finding target** — without it, BFS would explore neighbors of the destination unnecessarily
8. **`defaultdict(int)` for rate dict** — missing keys would return 0 instead of signaling absence; switched to regular `dict`

## Key Learnings

- **Store both directions during parsing** — precompute reverse rates (`1/rate`) to simplify lookup logic
- **BFS for all paths requires per-path visited sets** — shared visited set blocks valid alternative paths
- **Set mutation vs creation:** `set.add()` mutates in place (corrupts shared references); `set | {item}` or `set.copy()` creates a new set
- **All-paths BFS is O(V!)** — each permutation of intermediate nodes is a valid path. Fine for small graphs, mention scalability in interview
- **Don't over-engineer early parts** — Part 2 only needs one intermediate, simple loop suffices. Save BFS for Parts 3-4
- **Depth limiting in BFS** — add depth to queue tuple `(node, rate, depth)` and check `if depth < limit` before enqueuing

## Code Quality Notes

- Clean separation: `build_dict` for parsing, `getRate` for querying
- `build_dict` should be private (`_build_dict`) since it's an internal setup method
- String key pattern `f"{A}:{B}"` is simple but a tuple `(A, B)` would be more Pythonic and avoid string formatting overhead
- Could add `countryA == countryB` check returning 1.0 at the top of `getRate`

## Q&A Highlights

- **Why per-path visited sets?** Shared set means one path exploring EUR blocks another path from going through EUR — paths must be independent
- **`set.copy().add()` vs `set | {item}`** — both create new sets; `.copy()` + `.add()` is more explicit, `|` is more concise
- **Why is all-paths BFS O(V!)?** From source, V-2 intermediate choices, then V-3, then V-4... = (V-2)! paths. Each copies a visited set of size V, so O(V! × V) total work
- **BFS finds first path, not best path** — standard BFS returns shortest path by hop count, but for optimal exchange rate you need to explore ALL paths and track the maximum
