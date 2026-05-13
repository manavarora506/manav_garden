# Bank Balance Rebalancing

**Date:** 2026-02-13
**Category:** Financial Logic
**Parts Completed:** 3/3 (base + minimum moves + production readiness)
**Language:** Python

## Problem Summary

Given up to 500 bank accounts with varying balances, generate transfer instructions to bring all balances to at least 100. Follow-ups: minimize the number of moves, and discuss production readiness for real money movement.

## Solutions by Part

### Part 1: Basic Working Solution

**Approach:** Separate accounts into "need funds" (below 100) and "can give" (above 100) lists. Iterate through receivers, greedily pulling from givers. Transfer the minimum of what the giver can spare and what the receiver needs. Remove accounts from lists when they hit 100.

```python
from collections import defaultdict

BankAccounts = [
    {"country": "AU", "amount": 80},
    {"country": "US", "amount": 140},
    {"country": "MX", "amount": 110},
    {"country": "SG", "amount": 120},
    {"country": "FR", "amount": 70},
]

def moveBalances(BankAccounts: list):
    bank_to_amount = {account["country"]: account["amount"] for account in BankAccounts}

    banksThatNeedFunds = [country for country, amount in bank_to_amount.items() if amount < 100]
    banksThatCanGive = [country for country, amount in bank_to_amount.items() if amount > 100]

    while len(banksThatNeedFunds) > 0:
        bankRecieverCountry = banksThatNeedFunds[0]
        bankRecieverAmount = bank_to_amount[bankRecieverCountry]
        moneyNeeded = 100 - bankRecieverAmount
        while len(banksThatCanGive) > 0 and moneyNeeded > 0:
            bankGiverCountry = banksThatCanGive[0]
            bankGiverAmount = bank_to_amount[bankGiverCountry]
            bankGiverCanCover = bankGiverAmount - 100
            if bankGiverCanCover < moneyNeeded:
                bankWillTransfer = bankGiverCanCover
                moneyNeeded -= bankGiverCanCover
            else:
                bankWillTransfer = moneyNeeded
                moneyNeeded = 0
            if moneyNeeded == 0:
                banksThatNeedFunds.remove(bankRecieverCountry)
            bank_to_amount[bankGiverCountry] = bankGiverAmount - bankWillTransfer
            if bank_to_amount[bankGiverCountry] == 100:
                banksThatCanGive.remove(bankGiverCountry)
            print(f"from: {bankGiverCountry}, to: {bankRecieverCountry}, amount: {bankWillTransfer}")
```

### Part 2: Minimum Number of Moves

**Approach:** Pre-process exact matches where a giver's excess equals a receiver's deficit. Use two hashmaps (`deficit_banks` and `surplus_banks`) keyed by amount. Match exact pairs first (1 move removes 2 accounts), then fall back to Part 1's greedy loop for the rest.

```python
def moveBalances(BankAccounts: list):
    bank_to_amount = {account["country"]: account["amount"] for account in BankAccounts}

    deficit_banks = defaultdict(list)
    surplus_banks = defaultdict(list)

    for country, amount in bank_to_amount.items():
        diff = 100 - amount
        if diff < 0:
            surplus_banks[diff].append(country)
        else:
            deficit_banks[diff].append(country)

    # Exact match pre-processing
    for country, amount in bank_to_amount.items():
        diff = 100 - amount
        if diff < 0:
            if -diff in deficit_banks:
                receiver = deficit_banks[-diff].pop()
                surplus_banks[diff].remove(country)
                bank_to_amount[country] = 100
                bank_to_amount[receiver] = 100
                print(f"from: {country}, to: {receiver}, amount: {-diff}")
                if not deficit_banks[-diff]:
                    del deficit_banks[-diff]
                if not surplus_banks[diff]:
                    del surplus_banks[diff]

    # Greedy fallback for remaining accounts
    banksThatNeedFunds = [country for country, amount in bank_to_amount.items() if amount < 100]
    banksThatCanGive = [country for country, amount in bank_to_amount.items() if amount > 100]

    while len(banksThatNeedFunds) > 0:
        bankRecieverCountry = banksThatNeedFunds[0]
        bankRecieverAmount = bank_to_amount[bankRecieverCountry]
        moneyNeeded = 100 - bankRecieverAmount
        while len(banksThatCanGive) > 0 and moneyNeeded > 0:
            bankGiverCountry = banksThatCanGive[0]
            bankGiverAmount = bank_to_amount[bankGiverCountry]
            bankGiverCanCover = bankGiverAmount - 100
            if bankGiverCanCover < moneyNeeded:
                bankWillTransfer = bankGiverCanCover
                moneyNeeded -= bankGiverCanCover
            else:
                bankWillTransfer = moneyNeeded
                moneyNeeded = 0
            if moneyNeeded == 0:
                banksThatNeedFunds.remove(bankRecieverCountry)
            bank_to_amount[bankGiverCountry] = bankGiverAmount - bankWillTransfer
            if bank_to_amount[bankGiverCountry] == 100:
                banksThatCanGive.remove(bankGiverCountry)
            print(f"from: {bankGiverCountry}, to: {bankRecieverCountry}, amount: {bankWillTransfer}")
```

### Part 3: Production Readiness (Discussion)

**Approach:** Discussed how to harden this for moving real money.

- **Verification checks:**
  - Pre-check: total sum across all accounts is preserved and >= num_accounts * 100
  - Post-check: all balances >= 100, no negatives, total sum unchanged
- **Execution strategy:** Execute one move at a time (consistency over speed)
- **Failure handling:** Log all completed moves; on failure, recalculate remaining moves from current state and resume (rather than trying to reverse completed transfers)
- **Idempotency:** Each move gets a unique ID so retries don't double-transfer
- **Dry run:** Separate planning from execution — generate all moves, verify the plan, then execute
- **Alerting:** Every move logged with timestamp, amount, status; alert on failure

## Edge Cases

- Total balance across all accounts < num_accounts * 100 (no solution exists)
- Account exactly at 100 (neither giver nor receiver)
- Single giver covering multiple receivers (partial transfers)
- Single receiver needing funds from multiple givers
- All accounts already at or above 100 (no moves needed)

## Bugs & Issues

1. **Dict comprehension syntax** — used `{country, bank ...}` (set syntax) instead of `{key: value ...}` (dict syntax)
2. **`.keys()` vs `.items()`** — iterated over `.keys()` then tried to access `.amount` on strings
3. **Order of operations bug** — set `moneyNeeded = 0` before using it to update giver balance and print amount, resulting in zero transfers
4. **Missing partial transfer logic** — `else` branch only incremented index without transferring the partial amount the giver could cover
5. **Sign confusion in hashmaps** — mixed up which sign (positive/negative diff) corresponds to deficit vs surplus
6. **Modifying dict during iteration** — updated `bank_to_amount` values while iterating over `.items()`

## Key Learnings

- `min(moneyNeeded, excessAvailable)` is the core pattern for transfer problems — handles both partial and full transfers cleanly
- Exact-match pre-processing (hashmap lookup) reduces moves: an exact match removes 2 accounts in 1 move
- For production money movement: separate planning from execution, log everything, use idempotency keys, verify conservation of total
- Python dict iteration: `.keys()` gives keys only, `.items()` gives (key, value) tuples — know which you need

## Code Quality Notes

- Variable naming could be more Pythonic (snake_case over camelCase)
- `while len(list) > 0` can be simplified to `while list`
- Helper functions could separate concerns: `find_exact_matches()`, `generate_moves()`, `execute_moves()`
- The dict-modification-during-iteration pattern is risky; collecting matches first then updating is cleaner

## Q&A Highlights

- **"Is minimum moves greedy?"** — Not pure greedy or DP. The key insight is exact-match pairing (excess == deficit) via hashmaps
- **"Can exact matching ever be suboptimal?"** — No, an exact match always removes 2 accounts in 1 move, which is never worse than alternatives
- **"How to handle failed transfers?"** — Don't try to reverse completed transfers; instead log everything and recalculate remaining moves from current state
