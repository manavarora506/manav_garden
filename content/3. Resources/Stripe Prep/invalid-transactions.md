# Invalid Transactions

**Date:** 2026-02-22
**Category:** Data Processing
**Parts Completed:** 1/1
**Language:** Python

## Problem Summary

Given a list of transaction strings (name, time, amount, city), return all transactions that are possibly invalid. A transaction is invalid if the amount exceeds $1000, or if it occurs within (and including) 60 minutes of another transaction with the same name in a different city.

LeetCode 1169.

## Solution

**Approach:** Group transactions by name, then for each group compare every pair of transactions to check the time + city rule. Track invalid transactions by their original index to handle duplicate strings correctly. Check amount > 1000 independently.

```python
from dataclasses import dataclass
from collections import defaultdict
from typing import List


@dataclass
class Transaction:
    name: str
    time: int
    amount: int
    city: str
    index: int


class Solution:
    def invalidTransactions(self, transactions: List[str]) -> List[str]:
        parsed = []
        for i, raw in enumerate(transactions):
            name, time, amount, city = raw.split(',')
            parsed.append(Transaction(name, int(time), int(amount), city, i))

        groups = defaultdict(list)
        for txn in parsed:
            groups[txn.name].append(txn)

        invalid = set()

        for name, txns in groups.items():
            for i, curr_txn in enumerate(txns):
                if curr_txn.amount > 1000:
                    invalid.add(curr_txn.index)

                for other_txn in txns[i + 1:]:
                    if abs(curr_txn.time - other_txn.time) <= 60 and curr_txn.city != other_txn.city:
                        invalid.add(curr_txn.index)
                        invalid.add(other_txn.index)

        return [transactions[i] for i in invalid]
```

## Edge Cases

- Amount exactly 1000 (valid) vs 1001 (invalid)
- Time difference exactly 60 minutes (invalid — "within and including")
- Negative time differences (handled with `abs()`)
- Duplicate transaction strings in input (handled by tracking indices, not strings)
- Transaction invalid for both reasons (amount > 1000 AND time/city rule)
- Single transaction with amount > 1000 and no other transactions to compare

## Bugs & Issues

1. **Adjacent-only comparison** — Initially only compared transaction `i` with `i+1`, missing non-adjacent invalid pairs. Fixed by using nested loop over all pairs.
2. **`break` after invalid amount** — Broke out of the loop for a name group after finding one bad amount, skipping remaining transactions. Fixed by removing break.
3. **Last transaction unchecked** — While loop condition `i < len - 1` meant the last transaction was never checked for amount. Fixed with `i < len`.
4. **Missing `abs()` on time diff** — Without sorting by time, `nextTime - currTime` could be negative. Fixed with `abs()`.
5. **Set of strings collapsed duplicates** — Identical transaction strings appeared once in set even if both were invalid. Fixed by storing indices in the set instead.
6. **Only marking one transaction in a pair** — Initially only added the compared-against transaction, not the current one. Fixed by adding both.
7. **Over-grouping by (name, city)** — Considered grouping by name AND city, which would separate the transactions that need to be compared. Recognized grouping by name alone is correct.

## Key Learnings

- **Group by what you need to compare within** — The rule compares transactions with the same name but different cities, so group by name only
- **Use indices for identity** — When input can contain duplicate strings, track by index to preserve multiplicity
- **Avoid double negation** — `not isValid()` where `isValid` returns `> 60` is confusing; directly check `<= 60` instead
- **Inline simple conditions** — Helper functions for one-line checks (`isInDiffCity`, `isValidAmount`) add indirection without clarity in an interview setting
- **Dataclass for readability** — `curr_txn.amount > 1000` is much clearer than `t1[2] > 1000`; takes 30 seconds to define and makes the rest self-documenting

## Code Quality Notes

- Dataclass gives named field access, eliminating magic tuple indices
- `txns[i + 1:]` avoids both self-comparison and redundant A-vs-B / B-vs-A checks
- Parse once upfront (convert strings to ints at parse time) instead of repeatedly casting
- O(n²) time in worst case (all transactions under one name), O(n) space

## Q&A Highlights

- **Q: Should I group by name and city?** A: No — you need to compare across cities, so grouping by city separates the things you need to compare.
- **Q: Is the boundary condition (exactly 60 min) correct?** A: Yes — traced through the double negation to confirm. But cleaner to write the condition directly as `<= 60`.
- **Q: Should I include the index in the tuple?** A: Yes, and store indices (not strings) in the invalid set to handle duplicate transaction strings.
