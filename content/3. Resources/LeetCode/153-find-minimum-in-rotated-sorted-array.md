# 153. Find Minimum in Rotated Sorted Array

**Date:** 2026-03-19
**Difficulty:** Medium
**Topics:** Binary Search, Array
**Link:** https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/

## Final Solution

```python
class Solution:
    def findMin(self, nums: List[int]) -> int:
        start = 0
        end = len(nums) - 1
        while start < end:
            mid = start + (end - start) // 2
            if nums[mid] > nums[end]:
                start = mid + 1
            else:
                end = mid
        return nums[start]
```

## Initial Issues

- Compared `nums[mid] > nums[mid - 1]` — `mid = 0` causes `nums[-1]` to wrap to last element in Python
- Used `start = mid` with `while start <= end` — infinite loops when `start == end == mid`
- Compared `nums[mid] > end` — `end` is an index, not a value; must be `nums[end]`

## Key Learnings

### Compare `nums[mid]` against `nums[end]`
- `nums[mid] > nums[end]` → rotation point is to the **right**, so `start = mid + 1` (mid can't be the min since something smaller is on the right)
- `nums[mid] <= nums[end]` → mid could be the min or it's to the left, so `end = mid` (keep mid in search space)

### `while start < end` vs `while start <= end`

| | `start <= end` | `start < end` |
|---|---|---|
| Goal | Find target or return not found | Narrow down to one answer |
| Ends when | Search space empty (`start > end`) | One element left (`start == end`) |
| Return | Inside loop (found) or after (-1) | After loop: `nums[start]` |
| Used for | Exact target search (704, 74) | Find min, boundary, first/last |
| Why not the other | N/A | `end = mid` when `start == end` → infinite loop |

### Key insight
- `start < end` guarantees `mid < end` (since `mid = (start+end)//2` rounds down), so `end = mid` always shrinks the window
- Answer always exists — just shrinking to size 1

## Complexity

- Time: O(log n)
- Space: O(1)
