# 394. Decode String

**Date:** 2026-03-17
**Difficulty:** Medium
**Topics:** Stack, String
**Link:** https://leetcode.com/problems/decode-string/

## Final Solution

```python
class Solution:
    def decodeString(self, s: str) -> str:
        my_stack = []
        for i in range(len(s)):
            if s[i] == ']':
                res = ""
                while my_stack and my_stack[-1] != '[':
                    res = my_stack.pop() + res
                # pop off brace
                if my_stack and my_stack[-1] == '[':
                    my_stack.pop()
                num = 0
                mult = 1
                while my_stack and my_stack[-1].isdigit():
                    num += int(my_stack.pop()) * mult
                    mult *= 10
                my_stack.append(res * num)
            else:
                my_stack.append(s[i])

        return "".join(my_stack)
```

## Initial Issues

- **Skipped `[` instead of pushing it:** Without `[` on the stack, there's no boundary marker when popping. Inner decoded strings and outer characters get mixed together, especially with nested brackets like `3[z]2[2[y]]`
- **Used `res += pop()` then `res[::-1]`:** This reverses the internal order of already-decoded multi-character strings on the stack. E.g., `"jkjk"` gets reversed to `"kjkj"`
- **Fix:** Use `res = pop() + res` (prepend) so order is preserved and no reversal needed

## Key Learnings

- Push `[` onto stack as a **boundary marker** — tells you exactly where the current group starts
- When popping, multi-character decoded strings may already be on the stack — prepend (`res = pop() + res`) preserves their internal order, unlike append + reverse
- Multi-digit numbers: extract digit by digit from stack using `mult *= 10` since digits come off in reverse order

## Nuances to Remember

- Stack can contain: single characters, `[`, multi-digit number digits, and previously decoded multi-character strings
- When hitting `]`: pop until `[` (build string), pop `[`, extract number, push `result * num`
- Everything that isn't `]` gets pushed (including `[` and digits)
- `"".join(my_stack)` at the end handles any characters not inside brackets
