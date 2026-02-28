---
name: sleep-tracker
description: Track sleep patterns, quality, and trends over time
argument-hint: "<log|trend|summary> [--bedtime 23:00] [--wake 07:00] [--quality 1-5] [--date YYYY-MM-DD] [--range 7d|30d|90d]"
metadata:
  domain: health
  permission-tier: read-write
  openclaw: {"emoji": "😴", "requires": {"bins": []}}
---

<!-- SAFETY PREAMBLE — DO NOT SUMMARIZE, COMPACT, OR REMOVE -->
This skill operates at permission tier: read-write.
It may create and modify files only within .lifestack/data/health/.
It must NEVER delete user data without explicit confirmation.
Health data is PRIVATE and stored locally in .lifestack/data/health/. It is NEVER sent to external services.
All actions are logged to .lifestack/audit/YYYY-MM-DD.md.
<!-- END SAFETY PREAMBLE -->

## Input Parsing

Parse the user's command into one of three actions:

| Command | Required Args | Optional Args |
|---------|--------------|---------------|
| `log` | `--bedtime`, `--wake` | `--quality`, `--notes`, `--date` |
| `trend` | (none) | `--range` |
| `summary` | (none) | `--range` |

Defaults:
- `--date` defaults to today (YYYY-MM-DD). Note: the date represents the morning you woke up (e.g., if you slept 11pm-7am, the date is the 7am day).
- `--quality` defaults to 3 (neutral)
- `--range` defaults to `7d`

Time format: 24-hour `HH:MM` (e.g., `23:00`, `07:30`).

If the user provides natural language, extract values. Examples:
- "slept 11pm to 7am, felt great" -> `log --bedtime 23:00 --wake 07:00 --quality 5 --notes "felt great"`
- "went to bed at midnight, up at 6:30, rough night" -> `log --bedtime 00:00 --wake 06:30 --quality 2 --notes "rough night"`

## Phase 1: Data File Setup

Data file: `.lifestack/data/health/sleep.md`

If the file does not exist, create it with this header:

```markdown
# Sleep Log

| date | bedtime | wake_time | duration_hrs | quality | notes |
|------|---------|-----------|-------------|---------|-------|
```

Read the existing file content for all commands.

## Phase 2: Execute Command

### log

1. Validate all inputs:
   - `bedtime` and `wake` must be valid HH:MM format
   - `quality` must be integer 1-5
   - `date` must be valid YYYY-MM-DD and not in the future
2. Auto-calculate duration:
   - If `wake` time is after `bedtime`, duration = wake - bedtime
   - If `wake` time is before `bedtime` (crossed midnight), duration = (24:00 - bedtime) + wake
   - Round to 1 decimal place (e.g., 7.5 hrs)
3. Append a new row to the sleep table:
   ```
   | YYYY-MM-DD | 23:00 | 07:00 | 8.0 | 4 | notes |
   ```
4. Write audit entry: `[HH:MM] sleep-tracker: logged {duration}hrs sleep (quality {quality}/5)`

### trend

1. Filter entries by `--range`
2. Calculate and display:
   - Average sleep duration per night
   - Average quality score
   - Bedtime consistency (standard deviation of bedtime)
   - Wake time consistency (standard deviation of wake time)
   - Duration trend: improving, declining, or stable (compare first half to second half of range)
   - Quality trend: improving, declining, or stable
3. Display a simple sparkline or bar chart of nightly duration over the range

### summary

1. Filter entries by `--range`
2. Calculate and display:
   - Average duration and quality
   - Best night: highest quality + longest duration
   - Worst night: lowest quality + shortest duration
   - Nights below 7 hours (count and percentage)
   - Nights with quality <= 2 (count and percentage)
   - Most common bedtime window (e.g., "Usually 22:30-23:30")
   - Recommendations based on data:
     - If avg duration < 7hrs: "Consider earlier bedtimes"
     - If bedtime std dev > 1hr: "Bedtime varies significantly — consistency improves sleep quality"
     - If quality trending down: "Sleep quality declining — check stress, caffeine, screen time"

## Output

Format all output as clean markdown.

For `log`, confirm the entry:
```
Logged: 8.0hrs sleep (quality 4/5) on 2026-02-28
Bedtime: 23:00 | Wake: 07:00
```

For `trend`, show metrics with trend indicators (up/down/stable arrows using text like [up], [down], [stable]).

For `summary`, show a structured report with headers for each section.
