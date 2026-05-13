# Alert Using Same Key-Card Three or More Times in a One Hour Period

**Date:** 2026-02-22
**Category:** Data Processing / Rate Limiting
**Parts Completed:** 1/1
**Language:** Python

## Problem Summary

Given parallel arrays of worker names and keycard usage times ("HH:MM"), find all workers who used their keycard 3+ times within a one-hour window (inclusive, e.g. 10:00–11:00 is valid). Return names sorted alphabetically. LeetCode 1604.

## Solution

**Approach:** Group times by name, convert "HH:MM" to total minutes, sort each group, then sliding window — for each time, check how many subsequent times fall within +60 minutes. If 3+, flag the person.

```python
class Solution:
    def alertNames(self, keyName: List[str], keyTime: List[str]) -> List[str]:
        name_to_key_time = defaultdict(list)
        alerted_names = set()

        def convertTimeIntoMinutes(time: str):
            hours, mins = time.split(":")
            return int(hours) * 60 + int(mins)

        for name, time in zip(keyName, keyTime):
            minutes = convertTimeIntoMinutes(time)
            name_to_key_time[name].append(minutes)
        for name in name_to_key_time:
            name_to_key_time[name].sort()

        for name in name_to_key_time:
            keyTimes = name_to_key_time[name]
            i = 0
            while i < len(keyTimes):
                currTime = keyTimes[i]
                currWindow = currTime + 60
                numAlerts = 1
                j = i + 1
                while j < len(keyTimes) and keyTimes[j] <= currWindow:
                    numAlerts += 1
                    if numAlerts == 3:
                        alerted_names.add(name)
                        break
                    j += 1
                i += 1

        return sorted(list(alerted_names))
```

## Edge Cases

- Exactly 60 minutes apart is within the window (10:00 to 11:00 is valid)
- 22:51 to 23:52 is NOT within one hour (61 minutes)
- Person with fewer than 3 keycard uses — can never trigger alert
- Multiple groups of 3+ within same person — only add name once (set handles this)
- Times not given in order — sorting handles this

## Bugs & Issues

No major bugs this session — approach was sound from the start. Minor issues:
1. Initially considered converting times to floats (e.g. 1.30) — would give wrong arithmetic for minute differences. Converting to total minutes is correct.
2. Inner `break` only exits innermost loop — discussed using a `found` flag to skip remaining checks for an already-alerted person (optimization, not correctness).

## Key Learnings

- **Group → Sort → Sliding window** — core pattern for time-based frequency problems (rate limiting, fraud detection, log analysis)
- **Convert messy formats early** — parse "HH:MM" to total minutes at ingestion, then work with clean integers throughout
- **Sorted + consecutive check** — if any K items satisfy a window condition, after sorting there must be K consecutive items satisfying it. Avoids checking all combinations.
- **Don't use floats for time** — "01:30" as 1.30 float breaks arithmetic. Total minutes is the right representation.
- **Unpack in split** — `hours, mins = time.split(":")` is cleaner than splitting then indexing

## Code Quality Notes

- Clean separation: parsing → grouping → sorting → detection → output
- `defaultdict(list)` for natural grouping
- `set` for deduplication of alerted names
- Could add early exit with `found` flag to skip remaining times once a person is flagged

## Q&A Highlights

- **Q: Should I convert to floats?** A: No — "01:30" as 1.30 gives wrong differences. Convert to total minutes (hours * 60 + mins).
- **Q: How to sort each key's values?** A: Either sort after building dict, or convert+append then sort. Converting at append time means one pass through data.
- **Q: Does `break` exit both loops?** A: No — only the innermost. Use a `found` flag to also stop the outer loop.
