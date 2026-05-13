# Data Verification (Review Session)

**Date:** 2026-03-25
**Category:** Parsing / Data Processing
**Parts Completed:** 5/5 (review)
**Language:** Python

## Problem Summary

Business validation system that checks CSV records against multiple criteria: blocked terms, business name consistency across descriptors, and length requirements.

## Key Bugs Found on Review

### 1. Missing `continue` after short array check
```python
if len(business_arr) < 6:
    result.append(f"NOT VERIFIED: {business_name}")
    # Was missing continue — fell through to next if, causing double entries
    continue
```

### 2. Case sensitivity in blocked terms check
```python
# Bug: checked uppercase term against original-case descriptor
any(term.upper() in full_statement_descriptor for term in self.blocked_terms)
# Fix: uppercase both sides
any(term.upper() in full_statement_descriptor.upper() for term in self.blocked_terms)
```

### 3. Union vs separate check for col4/col5 matching
- Spec says "50% of col2 words must appear in **either** col4 or col5"
- Original code checked col4 and col5 independently with OR
- Fix: union the word sets, then check business words against the union
```python
total_words = full_statement_descriptor_words | short_statement_descriptor_words
match_count = sum(word in total_words for word in business_words)
```

### 4. Matching direction — semantic clarity
- `sum(word in total_words for word in business_words)` — iterate business words, check against descriptors
- Not `sum(word in business_words for word in total_words)` — same result for unique words but semantically backwards
- In an interview, loop direction should match the spec language

### 5. Division by zero guard
- If `business_words` is empty (name is all ignore words like "LLC INC"), divides by zero
- Fix: `if not business_words: return False`

## Key Learnings

- Always trace through code with concrete examples before declaring it correct
- Loop direction should match the English spec — shows precise translation of requirements
- Union vs separate check is a common spec interpretation gotcha
- Guard against empty collections before division
