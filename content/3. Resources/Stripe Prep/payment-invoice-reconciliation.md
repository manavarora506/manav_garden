# Payment Invoice Reconciliation

**Date:** 2026-02-09
**Category:** Parsing / Financial Logic
**Parts Completed:** 3/3
**Language:** Python
**Time:** ~36 minutes (4:36 → 5:12)

## Problem Summary

Build a payment reconciliation system that matches incoming payments to outstanding invoices. Parse comma-separated strings for payments and invoices, then match using a three-tier priority system: invoice ID in memo → exact amount match → fuzzy amount match within a forgiveness range. When multiple matches exist in the same tier, select the earliest due date.

## Solutions by Part

### Part 1: ID-Based Matching

**Approach:** Parse the payment string, check if memo contains ":" (indicating an invoice reference like "Paying for: inv-123"). Extract the invoice ID, compare case-insensitively against all invoices.

### Part 2: Amount-Based Matching

**Approach:** If no ID match from Part 1, sort invoices by due date and find the first invoice with an exact amount match. YYYY-MM-DD format sorts lexicographically, so string comparison works.

### Part 3: Fuzzy Amount Matching

**Approach:** If no exact match, iterate through date-sorted invoices and find the first where `abs(invoice_amount - payment_amount) <= forgiveness`.

### Combined Solution (All Parts)

```python
def reconcile_payment(payment: str, invoices: list, forgiveness: int = 0):
    payment_arr = payment.split(",")
    payment_id = payment_arr[0].strip()
    payment_amount = payment_arr[1].strip()
    memo = payment_arr[2].strip()

    # Tier 1: ID-based matching
    if ":" in memo:
        memo_arr = memo.split(':')
        invoice_id = memo_arr[1].lower().strip()
        for invoice in invoices:
            curr_invoice_arr = invoice.split(',')
            curr_invoice_id = curr_invoice_arr[0].strip()
            date = curr_invoice_arr[1].strip()
            amount = curr_invoice_arr[2].strip()
            if curr_invoice_id == invoice_id:
                return f"Payment {payment_id} paid {amount} for invoice {invoice_id} due on {date}"

    # Sort invoices by date for earliest-first selection
    sorted_invoices = sorted(invoices, key=lambda x: x.split(", ")[1])

    # Tier 2: Exact amount matching
    for sorted_invoice in sorted_invoices:
        curr_invoice_arr = sorted_invoice.split(',')
        curr_invoice_id = curr_invoice_arr[0].strip()
        date = curr_invoice_arr[1].strip()
        amount = curr_invoice_arr[2].strip()
        if amount == payment_amount:
            return f"Payment {payment_id} paid {amount} for invoice {curr_invoice_id} due on {date}"

    # Tier 3: Fuzzy amount matching
    for sorted_invoice in sorted_invoices:
        curr_invoice_arr = sorted_invoice.split(',')
        curr_invoice_id = curr_invoice_arr[0].strip()
        date = curr_invoice_arr[1].strip()
        amount = curr_invoice_arr[2].strip()
        if abs(int(amount) - int(payment_amount)) <= forgiveness:
            return f"Payment {payment_id} paid {payment_amount} for invoice {curr_invoice_id} due on {date}"

    return f"Payment {payment_id} could not be matched to any invoice"
```

## Edge Cases

- **Invoice ID in memo doesn't exist** → falls through to amount matching
- **No exact or fuzzy match** → returns "could not be matched"
- **Multiple invoices with same amount** → earliest due date wins (via sort)
- **Forgiveness boundary** → `abs(diff) <= forgiveness` includes exact boundary
- **Case sensitivity in memo** → `.lower()` on extracted invoice ID

## Bugs & Issues

- No major bugs in the final solution
- String comparison for amounts works for this problem but `int()` conversion would be more robust for general use
- Parsing could be extracted into a helper to avoid repeated `split`/`strip` logic (DRY)

## Key Learnings

- **Three-tier matching priority** is a common pattern: specific match → exact match → fuzzy match
- **YYYY-MM-DD sorts lexicographically** — no need for date parsing, string comparison works
- **Sort once, iterate multiple times** — sort invoices by date upfront, then the first match in each tier is automatically the earliest
- **Parse once, use many times** — could DRY up the invoice parsing with a helper function

## Code Quality Notes

- Invoice parsing is repeated three times — should extract a `parse_invoice()` helper
- Sorting invoices once upfront is smart — avoids tracking "earliest date" manually
- Could use `int()` for amount comparisons throughout for consistency (Tier 2 compares strings, Tier 3 compares ints)
- The `":" in memo` check is a simple heuristic — works but could be more robust with specific patterns like "paying for:" or "paying off:"

## Q&A Highlights

- **Why sort by date?** Instead of tracking the earliest date manually, sorting upfront means the first match in any tier is automatically the earliest-dated invoice
- **String vs int for amount comparison?** Tier 2 uses string comparison (`amount == payment_amount`) while Tier 3 uses int — should be consistent
