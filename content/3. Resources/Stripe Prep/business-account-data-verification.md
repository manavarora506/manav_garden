# Business Account Data Verification

**Date:** 2026-02-09
**Category:** Parsing / Data Processing
**Parts Completed:** 5/5
**Language:** Python

## Problem Summary

Build a KYC verification system that processes business accounts in CSV format. Validate accounts through progressively complex rules: field completeness, descriptor length constraints, generic name blocklist, name consistency checks, and specific error code classification.

## Solutions by Part

### Part 1: Complete Field Validation

**Approach:** Parse CSV by splitting on newlines, skip header, split each row by comma, use `all()` to check every field is non-empty after stripping whitespace.

### Part 2: Descriptor Length Validation

**Approach:** After field completeness check, verify col5 (full descriptor) length is between 5 and 31 inclusive using chained comparison `5 <= len(descriptor) <= 31`.

### Part 3: Generic Name Blocklist

**Approach:** Define blocked terms in a set. Use `any(term in descriptor.upper() for term in BLOCKED_TERMS)` for case-insensitive substring matching against col5.

### Part 4: Name Consistency Check

**Approach:** Extract words from col2, col4, and col5, filter out "LLC"/"Inc" (case-insensitive), combine descriptor words into one set, count matches from business name words. Verify >= 50% match rate.

### Part 5: Error Code Classification

**Approach:** Replace single if/else with if/elif chain checking validations in priority order. Each branch returns a specific error code. Only reaches VERIFIED if all checks pass.

### Combined Final Solution

```python
def validate_business_name(business_name, long_descriptor, short_descriptor):
    IGNORED_WORDS = ["llc", "inc"]

    def get_words(col, IGNORED_WORDS):
        words = col.split(" ")
        return [word.lower() for word in words if word.lower() not in IGNORED_WORDS]

    business_words = get_words(business_name, IGNORED_WORDS)
    if not business_words:
        return False

    descriptor_words = set(get_words(long_descriptor, IGNORED_WORDS) + get_words(short_descriptor, IGNORED_WORDS))
    matches = [w for w in business_words if w in descriptor_words]

    return len(matches) / len(business_words) >= 0.5


def validate_businesses(csv_data: str):
    csv_data_arr = csv_data.split('\n')

    BLOCKED_TERMS = set(["ONLINE STORE", "ECOMMERCE", "RETAIL", "SHOP", "GENERAL MERCHANDISE"])
    for i, row in enumerate(csv_data_arr):
        if i == 0:
            continue
        row_arr = row.split(',')
        business_name = row_arr[1].strip() if row_arr[1] else ""
        complete = all(field.strip() for field in row_arr)

        long_descriptor = row_arr[4].strip()
        short_descriptor = row_arr[3].strip()

        business_name_valid = validate_business_name(business_name, long_descriptor, short_descriptor)
        meets_length_requirement = (5 <= len(long_descriptor) <= 31)
        descriptor_contains_blocked_term = any(term in long_descriptor.upper() for term in BLOCKED_TERMS)

        if not complete:
            print(f"ERROR_MISSING_FIELDS: {business_name}")
        elif not meets_length_requirement:
            print(f"ERROR_INVALID_LENGTH: {business_name}")
        elif descriptor_contains_blocked_term:
            print(f"ERROR_GENERIC_NAME: {business_name}")
        elif not business_name_valid:
            print(f"ERROR_NAME_MISMATCH: {business_name}")
        else:
            print(f"VERIFIED: {business_name}")
```

## Edge Cases

- **Empty business name (col2)** → output shows empty after the colon
- **Fields with only whitespace** → treated as empty by `field.strip()`
- **Descriptor exactly at boundaries** → 5 and 31 are valid (inclusive)
- **"SHOP" inside "WORKSHOP"** → current substring matching would flag it (clarify with interviewer)
- **Business name is just "LLC Inc"** → zero words after filtering, returns False
- **Division by zero** → guarded by `if not business_words: return False`

## Bugs & Issues

1. **Part 1:** Used `/n` instead of `\n` for newline character
2. **Part 1:** Used `row.split(',')` where `row` was an integer index, not the string
3. **Part 1:** Printing instead of returning — breaks composability
4. **Part 4:** List comprehension syntax wrong — `[word if condition]` instead of `[word for word in ... if condition]`
5. **Part 4:** Trailing `/` in float division expression
6. **Part 4:** Not lowering words before comparing against lowercase IGNORED_WORDS — "LLC" wouldn't match "llc"
7. **Part 4:** Passing already-split list to `get_words()` which expects a string
8. **Part 4:** `name_words` undefined — variable was named `business_words`
9. **Part 4:** `get_words` defined with 2 params but called with 1
10. **Part 4:** Unreachable `return False` after unconditional return statement
11. **Part 4:** `row[3]` instead of `row_arr[3]` — accessing raw string instead of split array
12. **Part 4:** Code indented inside `get_words` function instead of at function body level

## Key Learnings

- **`all()` and `any()` with generators** — clean Pythonic way to check conditions across collections
- **Chained comparisons** — `5 <= len(x) <= 31` is valid Python and reads naturally
- **if/elif chain for priority-ordered validation** — cleanly maps to "return first error" requirement
- **Combine sets for OR matching** — `set(list_a + list_b)` then check membership, instead of checking two lists separately
- **`word.lower() not in IGNORED_WORDS`** — must normalize before comparing against normalized list
- **Guard against empty collections before division** — always check before `len(x) / len(y)`

## Code Quality Notes

- Good extraction of `validate_business_name` as a separate function
- `get_words` helper reduces repetition for word normalization
- Could further clean up by making `IGNORED_WORDS` a module-level constant
- Should return results instead of printing for testability
- The `IGNORED_WORDS` parameter to `get_words` could use the closure instead of being passed explicitly

## Q&A Highlights

- **Can you do boolean conditions in comprehensions?** Yes — `all(condition for item in list)` and `any(condition for item in list)`
- **`ord()` not needed here** but was learned in the font renderer problem — good to have in the toolkit
- **Substring vs exact word matching for blocked terms** — current implementation uses substring (`term in string`), should clarify with interviewer whether "SHOP" in "WORKSHOP" should match
