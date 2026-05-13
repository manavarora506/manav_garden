# Bit Font Renderer

**Date:** 2026-02-09
**Category:** Parsing / Data Processing
**Parts Completed:** 2.5/3 (Part 3 in progress)
**Language:** Python

## Problem Summary

Build a text rendering system using bitmap fonts. Characters are grids of 0s and 1s rendered as `.` and `#`. Progressive parts: render a single character, render a word by combining characters horizontally, and support run-length encoded (RLE) fonts.

## Solutions by Part

### Part 1: Single Character Rendering

**Approach:** Replace `0` with `.` and `1` with `#` in each row. Return a new list rather than mutating input.

```python
def render_character(grid):
    result = []
    for i, row in enumerate(grid):
        rendered_row = ""
        for cell in row:
            if cell == "0":
                rendered_row += "."
            elif cell == "1":
                rendered_row += "#"
        result.append(rendered_row)
    return result
```

One-liner alternative: `return [row.replace("0", ".").replace("1", "#") for row in grid]`

### Part 2: Word Rendering

**Approach:** For each character in the text, concatenate its binary rows horizontally (row by row). After building the combined binary grid, apply `render_character` for the visual conversion.

```python
def render_word(text, font):
    if not text:
        return []
    chars = font["chars"]
    height = len(chars[text[0]])
    result = ["" for i in range(height)]
    for ch in text:
        letter = chars[ch]
        for i, row in enumerate(letter):
            result[i] += row
    result = render_character(result)
    return result
```

### Part 3: RLE Decoding (In Progress)

**Approach:** Decode each encoded row by iterating through each character, converting to a run length (digits 0-9 directly, letters a-z map to 10-35 via `ord(c) - ord('a') + 10`). Runs alternate off/on starting with off. Reset off state at the start of each row. Then combine decoded characters horizontally using Part 2 logic.

```python
def char_to_length(c):
    if c.isdigit():
        return int(c)
    else:
        return ord(c) - ord('a') + 10

def decode_rle(encoded_rows):
    result = []
    for es in encoded_rows:
        off = True
        decoded_str = ""
        for ele in es:
            if off:
                decoded_str += "0" * char_to_length(ele)
            else:
                decoded_str += "1" * char_to_length(ele)
            off = not off
        result.append(decoded_str)
    return result

def render_word_rle(text, font):
    if not text:
        return []
    chars = font["chars"]
    height = len(chars[text[0]])
    result = ["" for i in range(height)]
    for ch in text:
        decoded_rows = decode_rle(chars[ch])
        for i, row in enumerate(decoded_rows):
            result[i] += row
    result = render_character(result)
    return result
```

## Edge Cases

- **Empty grid** → return empty list
- **Empty text** → return empty list
- **Variable width characters** → handled by concatenating each character's rows independently
- **Characters not in font dict** → would throw KeyError, could add a check
- **RLE row with odd number of elements** → last run ends on whatever state it's in

## Bugs & Issues

1. **Part 1:** Mutated input grid in place instead of returning a new list
2. **Part 1:** Function printed instead of returning — breaks composability with Part 2
3. **Part 2:** Used loop-and-break to get height instead of `len(chars[text[0]])`
4. **Part 3:** Treated each encoded row as a single element instead of iterating through characters within the row — missing inner loop
5. **Part 3:** `off` state wasn't reset per row — carried over from previous row, corrupting alternation
6. **Part 3:** `result = []` was inside the row loop — reset on every iteration, only kept last row
7. **Part 3:** Initially didn't know `ord()` trick for converting letters to numbers (a=10, b=11, ..., z=35)
8. **Part 3:** Initially had off/on logic backwards — appending 1s when off and 0s when on

## Key Learnings

- **Don't mutate input data** — return new structures, especially in interview settings
- **Functions should return, not print** — enables composition and testing
- **`ord(c) - ord('a') + 10`** — standard pattern for mapping letters to numbers
- **Reset state per iteration** — when processing rows independently, state like `off` must reset each row
- **Variable scoping in loops** — placing `result = []` inside vs outside a loop is a common bug
- **Separate decode from render** — `decode_rle` as its own function keeps concerns clean and makes debugging easier

## Code Quality Notes

- Should extract `decode_rle` as a standalone function for testability
- `off = not off` is cleaner than tracking two separate booleans
- Could use `is_on` instead of `off` to avoid double-negative confusion
- Debug prints should be removed before submitting

## Q&A Highlights

- **When to apply 0→./1→# conversion?** After concatenation — combine binary strings first, render once at the end
- **How to convert 'a' to 10?** `ord('a') - ord('a') + 10` — `ord()` gives ASCII value, subtract base, add offset

---

## ⚠️ COME BACK TO THIS

**Part 3 not fully completed.** Need to:
- Finish integrating `decode_rle` with the horizontal combining logic (`render_word_rle`)
- Test end-to-end with the RLE font example
- Practice the three-level loop pattern: characters in text → rows per character → elements per encoded row
- The main struggle was understanding the loop nesting — practice similar string-decoding problems
