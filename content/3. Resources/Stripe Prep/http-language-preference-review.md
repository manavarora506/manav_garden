# HTTP Language Preference (Review Session)

**Date:** 2026-03-25
**Category:** Parsing
**Parts Completed:** 4/4 (review)
**Language:** Python

## Problem Summary

Parse Accept-Language HTTP headers with exact matching, prefix/generic matching, wildcard support, and q-factor weighting.

## Key Bugs Found on Review

### 1. Substring matching instead of prefix matching
```python
# Bug: "fr" in "afr-ZA" → True (false positive)
if region in lang
# Fix: prefix check
if lang.startswith(region + "-")
# Or: if lang.split("-")[0] == region
```

### 2. Set for supported languages loses ordering
```python
# Bug: iteration order non-deterministic
self.supported_languages = set(supported_languages)
# Fix: keep list for order, use set for lookups
self.supported_languages = supported_languages
self.supported_set = set(supported_languages)
```

### 3. No handling for missing q-factor
```python
# Bug: crashes on "en-US" (no ;q=)
lang, q_factor = q.split(';')  # ValueError
# Fix:
if ';' in q:
    lang, q_factor = q.split(';')
    q_val = float(q_factor.split("=")[1])
else:
    lang = q
    q_val = 1.0
```

### 4. q=0 languages appended to result
- HTTP spec says q=0 = "not acceptable"
- Check problem spec: some variants want them listed last, others excluded entirely

## Key Learnings

- Substring vs prefix matching is a classic Stripe interview trap — always use startswith
- Sets lose ordering — use lists when order matters, sets for O(1) lookups
- HTTP spec: missing q-factor = 1.0, q=0 = not acceptable
- Always handle the "missing optional parameter" case
