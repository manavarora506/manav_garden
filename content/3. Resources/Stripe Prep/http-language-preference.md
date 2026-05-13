# HTTP Request Language Preference

**Date:** 2026-02-06
**Category:** Parsing
**Parts Completed:** 3/4
**Language:** Python

## Problem Summary

Build a content delivery system that determines which languages to serve based on the HTTP Accept-Language header. The problem has 4 progressive parts: exact matching, prefix matching, wildcard support, and quality factor weighting.

## Solutions by Part

### Part 1: Exact Language Tag Matching

**Approach:** Parse comma-separated header into list, use a set for O(1) lookup of supported languages, iterate header in order to maintain preference.

```python
def parse_accept_language(accept_header, supported_languages):
    result = []
    supported_set = set(supported_languages)
    accepted_languages = [lang.strip() for lang in accept_header.split(",")]
    for language in accepted_languages:
        if language in supported_set:
            result.append(language)
    return result
```

### Part 2: Generic Language Matching

**Approach:** Added prefix matching - if a tag has no hyphen, use `startswith()` to match all supported variants. Used a `seen` set for O(1) dedup. Single pass over header to preserve preference order.

```python
def parse_accept_language(accept_header, supported_languages):
    result = []
    seen = set()

    def parse_header(accept_header):
        return [language.strip() for language in accept_header.split(",")]

    supported_set = set(supported_languages)
    accepted_languages = parse_header(accept_header)
    for language in accepted_languages:
        if language in supported_set and language not in seen:
            seen.add(language)
            result.append(language)
        elif "-" not in language:
            for sup_language in supported_languages:
                if sup_language.startswith(language + "-") and sup_language not in seen:
                    seen.add(sup_language)
                    result.append(sup_language)
    return result
```

### Part 3: Wildcard Support

**Approach:** Added `*` handling as another elif branch - when encountered, add all unseen supported languages. Key bug caught: `*` has no hyphen so it was falling into the prefix branch before the wildcard check.

```python
def parse_accept_language(accept_header, supported_languages):
    result = []
    seen = set()

    def parse_header(accept_header):
        return [language.strip() for language in accept_header.split(",")]

    supported_set = set(supported_languages)
    accepted_languages = parse_header(accept_header)
    for language in accepted_languages:
        if language in supported_set and language not in seen:
            seen.add(language)
            result.append(language)
        elif language == "*":
            for sup_language in supported_languages:
                if sup_language not in seen:
                    seen.add(sup_language)
                    result.append(sup_language)
        elif "-" not in language and language != "*":
            for sup_language in supported_languages:
                if sup_language.startswith(language + "-") and sup_language not in seen:
                    seen.add(sup_language)
                    result.append(sup_language)
    return result
```

### Part 4: Quality Factor Weighting (Not Attempted)

Parses q-factors (e.g., `en-US;q=0.8`) and sorts by weight descending. Left for next session.

## Edge Cases

- Whitespace around language tags after splitting on commas
- Duplicate languages across exact and prefix matches (e.g., `"fr-FR, fr"`)
- Wildcard `*` falling into prefix branch due to no hyphen
- Set iteration order being non-deterministic for prefix matches
- Empty header / no matches returning empty list

## Bugs & Issues

- **Two-pass ordering bug:** Initial Part 2 solution used separate loops for exact and prefix matching, which broke preference order when prefixes appeared before exact matches in the header
- **Wildcard condition ordering:** `*` has no hyphen, so `elif '-' not in language` caught it before the wildcard check. Fixed by either reordering conditions or adding `language != '*'` guard

## Key Learnings

- **Single-pass preserves order:** Doing exact, prefix, and wildcard matching in one loop over the header naturally maintains preference order
- **`startswith(prefix + "-")`** is cleaner than splitting on hyphen and comparing parts
- **Separate `seen` set** from `result` list gives O(1) dedup while preserving insertion order
- **`set(supported_languages)`** is more idiomatic than set comprehension `{x for x in list}`
- **Condition ordering matters** in elif chains - more specific checks should come first

## Code Quality Notes

- Extracted `parse_header` as a helper early, anticipating parsing complexity in Part 4
- Variable naming improved from `accepted_langauges` (typo) to `accepted_languages`
- Debug `print()` statements should be removed before moving on
- Could iterate list instead of set in inner loops for deterministic ordering

## Q&A Highlights

- **Q:** Is there a strip/trim method in Python? **A:** Yes - `.strip()` which was already being used. Also `.lstrip()` and `.rstrip()` for directional trimming.
- **Q:** Should parsing be done in one pass or two? **A:** One pass over header tags, resolving all matches (exact, prefix, wildcard) per tag, preserves preference order naturally.
- **Q:** Can input have multiple language codes? **A:** Yes, comma-separated. Individual tags follow `language-REGION` format.
