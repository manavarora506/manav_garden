# Account Balance Manager

**Date:** 2026-02-09
**Category:** Financial Logic
**Parts Completed:** 3/3
**Language:** Python

## Problem Summary

Build an account balance management system for a payment platform. Process financial transactions across multiple accounts with progressive complexity: basic aggregation, transaction validation (reject if negative), and platform account coverage (cover shortfalls from a designated platform account).

## Solutions by Part

### Part 1: Balance Aggregation

**Approach:** Use a defaultdict to accumulate balances per account. Filter output to only include accounts with positive balances.

```python
from collections import defaultdict

def get_account_balances(transactions):
    balance = defaultdict(int)
    for transaction in transactions:
        account_id, amount = transaction["account_id"], transaction["amount"]
        balance[account_id] += amount
    return {account_id: amount for account_id, amount in balance.items() if amount > 0}
```

**Time Complexity:** O(n)

### Part 2: Transaction Validation

**Approach:** Check if a withdrawal would cause negative balance before applying it. If it would, reject the transaction (don't apply). Track rejected transactions in a list. Return both balances and rejected list. Changed filter from `> 0` to `>= 0` since Part 2 includes zero-balance accounts.

```python
from collections import defaultdict

def process_transactions(transactions):
    balance = defaultdict(int)
    rejected_transactions = []
    for transaction in transactions:
        account_id, amount = transaction["account_id"], transaction["amount"]
        curr_balance = balance[account_id]
        if curr_balance + amount < 0:
            rejected_transactions.append(transaction)
        else:
            balance[account_id] += amount
    return ({account_id: amount for account_id, amount in balance.items() if amount >= 0}, rejected_transactions)
```

**Time Complexity:** O(n)

### Part 3: Platform Account Coverage

**Approach:** Instead of rejecting, cover the shortfall from a platform account. Calculate exact shortfall as `-(curr_balance + amount)`, deduct from platform, set account balance to 0. Guard against the platform account covering itself. Treat platform as a normal account in the balance dict so it handles multiple deposits naturally.

```python
from collections import defaultdict

def process_with_coverage(transactions, platform_account_id):
    coverage_amount = 0
    balance = defaultdict(int)
    
    for transaction in transactions:
        account_id, amount = transaction["account_id"], transaction["amount"]
        curr_balance = balance[account_id]
        if account_id != platform_account_id and curr_balance + amount < 0:
            diff = -(curr_balance + amount)
            balance[platform_account_id] -= diff
            coverage_amount += diff
            balance[account_id] = 0
        else:
            balance[account_id] += amount

    return coverage_amount
```

**Time Complexity:** O(n)

## Edge Cases

- **Empty transaction list** → return empty dict / no rejections / 0 coverage
- **All accounts end at zero** → Part 1 excludes them, Part 2 includes them
- **Account with only rejected withdrawals** → never enters balance dict (unless defaultdict accessed)
- **Platform account going negative** → guarded by `account_id != platform_account_id` check
- **Multiple platform deposits** → handled naturally since platform is just another account in the dict
- **Deposits are always valid** → positive amounts can never cause negative balance

## Bugs & Issues

1. **Part 1:** Import should be `from collections import defaultdict`, not `import defaultdict`
2. **Part 2:** Initially kept `> 0` filter from Part 1, but Part 2 requires including zero-balance accounts (`>= 0`)
3. **Part 3:** Initially searched for platform account and skipped it — overcomplicated; simpler to treat it as a normal account with a guard condition
4. **Part 3:** Coverage calculation was `abs(curr_balance - amount)` instead of `-(curr_balance + amount)` — subtraction vs addition error
5. **Part 3:** Forgot to set `balance[account_id] = 0` after coverage — caused stale balance on subsequent transactions
6. **Part 3:** Missing platform self-coverage guard — platform could try to cover itself on negative withdrawal

## Key Learnings

- **Treat special accounts uniformly** — the platform account should flow through the same balance dict as every other account. Don't special-case it with index searching.
- **Coverage = -(balance + amount)** — the shortfall is how far below zero the new balance would be. Easy to get the sign wrong.
- **Set balance to 0 after coverage** — forgetting this causes cascading bugs on later transactions for the same account.
- **Read requirement changes carefully between parts** — Part 1 excludes zero balances, Part 2 includes them. Small detail, easy to miss.

## Code Quality Notes

- Clean, consistent structure across all three parts — each builds naturally on the previous
- Good use of defaultdict to avoid key-existence checks
- Dict comprehension for filtering is Pythonic and readable
- Could extract the transaction unpacking into a helper if more fields were added

## Q&A Highlights

- **Why `>= 0` in Part 2 vs `> 0` in Part 1?** Requirements changed — Part 2 explicitly says "include accounts with zero balance"
- **Why not search for the platform account?** It could have multiple transactions; treating it as a normal account in the dict handles all cases
- **Why guard against platform self-coverage?** Without the guard, a platform withdrawal going negative would trigger coverage from itself — nonsensical
