# Subscription Email Scheduler

**Date:** 2026-02-16
**Category:** Data Processing / State Machine
**Parts Completed:** 2/3 (Part 3 in progress)
**Language:** Python

## Problem Summary

Build an email notification system for a subscription service. Generate scheduled emails at lifecycle events (welcome, upcoming expiration, expired). Progressive parts add plan changes and subscription renewals that modify future notifications.

## Solutions by Part

### Part 1: Basic Email Scheduling

**Approach:** Use a defaultdict mapping day → list of email events. For each user, iterate the send_schedule and calculate notification dates: "start" → account_date, "end" → end_date, numeric keys → end_date + offset. Sort by day and print.

```python
from collections import defaultdict

def send_emails(user_accounts, send_schedule):
    day_to_msgs = defaultdict(list)

    for user in user_accounts:
        name, account_date, duration, plan = user["name"], user["account_date"], user["duration"], user["plan"]
        end_date = account_date + duration
        for key, message_type in send_schedule.items():
            if key == "start":
                notification_date = account_date
            elif key == "end":
                notification_date = end_date
            else:
                offset = int(key)
                notification_date = end_date + offset

            day_to_msgs[notification_date].append({"name": name, "plan": plan, "message_type": message_type})

    for day in sorted(day_to_msgs.keys()):
        for email in day_to_msgs[day]:
            print(f"{day}: [{email['message_type']}] Subscription for {email['name']} ({email['plan']})")
```

### Part 2: Plan Changes

**Approach:** Add a `name_to_plan` dict tracking each user's current plan. Don't bake plan names into notification events — look them up at print time. Add plan change events to the same `day_to_msgs` timeline. Sort within each day so "Changed" events process before notifications (using `key=lambda x: x["message_type"] != "Changed"`). Update `name_to_plan` when processing a change event.

```python
from collections import defaultdict

def send_emails(user_accounts, send_schedule, plan_changes):
    day_to_msgs = defaultdict(list)
    name_to_plan = {account["name"]: account["plan"] for account in user_accounts}

    for user in user_accounts:
        name, account_date, duration, plan = user["name"], user["account_date"], user["duration"], user["plan"]
        end_date = account_date + duration
        for key, message_type in send_schedule.items():
            if key == "start":
                notification_date = account_date
            elif key == "end":
                notification_date = end_date
            else:
                offset = int(key)
                notification_date = end_date + offset

            day_to_msgs[notification_date].append({"name": name, "message_type": message_type})

    for plan_change in plan_changes:
        name, new_plan, change_date = plan_change["name"], plan_change["new_plan"], plan_change["change_date"]
        day_to_msgs[change_date].append({"name": name, "plan": new_plan, "message_type": "Changed"})

    for day in sorted(day_to_msgs.keys()):
        day_to_msgs[day].sort(key=lambda x: x["message_type"] != "Changed")
        for email in day_to_msgs[day]:
            if email['message_type'] == "Changed":
                name_to_plan[email["name"]] = email["plan"]
            print(f"{day}: [{email['message_type']}] Subscription for {email['name']} ({name_to_plan[email['name']]})")
```

### Part 3: Subscription Renewals (In Progress)

**Approach (discussed, not yet coded):**
- Distinguish renewals vs plan changes by checking `"new_plan" in change` vs `"extension" in change`
- Track `name_to_end` dict for each user's current end date
- On renewal: update end date (`name_to_end[name] += extension`), add "Renewed" event, generate new notifications based on new end date
- Tag each notification with the `end_date` it was calculated from
- At print time, skip notifications whose stored `end_date` doesn't match the user's current end date (filters stale notifications without modifying dict during iteration)
- "Welcome" notifications use account_date not end_date, so they should never be filtered

## Edge Cases

- Notification date falls before subscription start (e.g., -15 on a 15-day sub → lands on start date)
- Multiple emails on the same day for the same user
- Plan change and notification on the same day (change must process first)
- Renewal changes end date, invalidating previously calculated offset notifications
- "Welcome" notification should never be filtered by end_date checks

## Bugs & Issues

1. **defaultdict syntax** — used `defaultdict[list]` instead of `defaultdict(list)`
2. **Loop variable name** — `for user in user` instead of `for user in user_accounts`
3. **Missing dict braces** — `.append("name": name, ...)` instead of `.append({"name": name, ...})`
4. **Nested quotes in f-string** — `name_to_plan[email["name"]]` inside double-quoted f-string; need single quotes inside
5. **Stale variable reference** — used `new_plan` from a previous loop instead of `email["plan"]`
6. **Print indentation** — print statement accidentally nested inside `if` block
7. **Same-day sort logic inverted** — `x["message_type"] == "Changed"` sorts Changed LAST (True=1); need `!=` to sort Changed FIRST

## Key Learnings

- **Chronological event processing** is powerful for state-dependent systems — merge all events into one timeline, track state as you go, look up current state at print time
- **Same-day ordering matters** — when events on the same day depend on each other (plan change before notification), explicit sorting is required
- **Don't bake derived state into events** — store the plan name in a lookup dict, not in each notification, so updates automatically propagate
- **Tag events with their source data** — storing which `end_date` a notification was calculated from enables filtering stale events without modifying the data structure during iteration
- **Boolean sort trick** — `key=lambda x: x != "target"` puts "target" first since False (0) < True (1)

## Code Quality Notes

- Good use of defaultdict for day → messages mapping
- `name_to_plan` lookup pattern cleanly separates mutable state from event data
- Part 3 approach of tagging + filtering avoids messy dict modification during iteration
- Could extract notification generation into a helper function for reuse in Part 3 renewals

## Q&A Highlights

- **Why not merge user_accounts and plan_changes?** They go into the same `day_to_msgs` dict from separate loops — the shared data structure is the merge point, not the input
- **How to handle renewals without modifying dict during iteration?** Tag each notification with the end_date it was calculated from; at print time, filter against the user's current end_date
- **Edge case with "Welcome" on renewals:** Welcome uses account_date not end_date, so it should bypass the end_date staleness filter
