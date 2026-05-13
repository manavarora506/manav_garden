# Payment Invoice Reconciliation (Review Session)

**Date:** 2026-03-25
**Category:** Parsing / Financial Logic
**Parts Completed:** 3/3 (review)
**Language:** Python

## Problem Summary

Match payments to invoices using three-tier matching: exact ID match from memo → amount match → fuzzy match with forgiveness threshold. Sorted by earliest date.

## Key Bugs Found on Review

### 1. `self.invoices` referenced before assignment
```python
# Bug: self.invoices doesn't exist yet
self.invoices = [inv.lower() for inv in self.invoices]
# Fix: use the parameter
self.invoices = [inv.lower() for inv in invoices]
```

### 2. Variable shadowing — `amount` reused
```python
payment_id, amount, memo = ...  # payment amount
curr_id, date, amount = ...     # overwrites with invoice amount!
# Fix: use distinct names (payment_amount, inv_amount)
```

### 3. `id` vs `inv_id` — using Python built-in
```python
f"... for invoice {id} ..."  # Python's built-in id() function!
# Fix: use curr_id or inv_id
```

### 4. Missing .strip() on extracted invoice ID
```python
inv_id = memo[pos + len("paying for:"):].strip()  # .strip() was missing
```

### 5. Duplicate amountMatch call instead of fuzzyMatch
```python
# Bug: called amountMatch twice
res = self.amountMatch(payment_id, amount)
res = self.amountMatch(payment_id, amount)  # should be fuzzyMatch
```

### 6. Fuzzy runs even when amount matches — needs elif
```python
# Bug: separate if statements, both can fire
# Fix: elif so fuzzy only runs when amount match fails
res = self.amountMatch(payment_id, amount)
if res:
    result.append(res)
elif self.forgiveness is not None:
    res = self.fuzzyMatch(payment_id, amount)
```

### 7. fuzzyMatch crashes when forgiveness is None
- Guard with `elif self.forgiveness is not None` before calling fuzzyMatch

### 8. Inconsistent split across methods
- fuzzyMatch used `split(", ")` while others used `split(",")` with strip
- Fix: use consistent `split(",")` + strip everywhere

## Key Learnings

- `self.x` vs parameter `x` in `__init__` is a common Python gotcha
- Variable shadowing across scopes is easy to miss — use distinct names
- Fallback tiers must use elif, not separate if statements
- Always guard against None before comparison operators
