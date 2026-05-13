# Python Nuances

## Nested Comprehensions — Loop Order (2026-02-18)

**Key takeaway:** In nested comprehensions, the for-loop order matches how you'd write them normally — outer loop first, inner loop second, left to right.

```python
# Nested comprehension
result = [x * y for x in range(3) for y in range(3)]

# Is equivalent to:
result = []
for x in range(3):       # outer loop (written first)
    for y in range(3):   # inner loop (written second)
        result.append(x * y)
```

The left-to-right reading order in the comprehension mirrors the top-to-bottom nesting of regular for loops.
