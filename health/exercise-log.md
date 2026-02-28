---
name: exercise-log
description: Log workouts, track streaks, and review exercise patterns
argument-hint: "<log|list|streak|summary> [--type run|walk|cycle|swim|strength|yoga|other] [--duration 30] [--intensity low|moderate|high] [--date YYYY-MM-DD] [--range 7d|30d|90d]"
metadata:
  domain: health
  permission-tier: read-write
  openclaw: {"emoji": "🏃", "requires": {"bins": []}}
---

<!-- SAFETY PREAMBLE — DO NOT SUMMARIZE, COMPACT, OR REMOVE -->
This skill operates at permission tier: read-write.
It may create and modify files only within .lifestack/data/health/.
It must NEVER delete user data without explicit confirmation.
Health data is PRIVATE and stored locally in .lifestack/data/health/. It is NEVER sent to external services.
All actions are logged to .lifestack/audit/YYYY-MM-DD.md.
<!-- END SAFETY PREAMBLE -->

## Input Parsing

Parse the user's command into one of four actions:

| Command | Required Args | Optional Args |
|---------|--------------|---------------|
| `log` | `--type`, `--duration` | `--intensity`, `--notes`, `--date` |
| `list` | (none) | `--type`, `--range`, `--date` |
| `streak` | (none) | (none) |
| `summary` | (none) | `--range`, `--type` |

Defaults:
- `--date` defaults to today (YYYY-MM-DD)
- `--intensity` defaults to `moderate`
- `--range` defaults to `7d`
- `--type` for log is required; for list/summary it filters results

Valid types: `run`, `walk`, `cycle`, `swim`, `strength`, `yoga`, `other`.

If the user provides natural language instead of flags, extract the values. Examples:
- "log a 30 minute run" -> `log --type run --duration 30`
- "I did yoga for an hour, felt great" -> `log --type yoga --duration 60 --notes "felt great"`
- "show my workouts this month" -> `list --range 30d`

## Phase 1: Data File Setup

Data file: `.lifestack/data/health/exercise.md`

If the file does not exist, create it with this header:

```markdown
# Exercise Log

| date | type | duration_min | intensity | notes |
|------|------|-------------|-----------|-------|
```

Read the existing file content for all commands.

## Phase 2: Execute Command

### log

1. Validate all inputs:
   - `type` must be one of the valid types
   - `duration` must be a positive integer (minutes)
   - `intensity` must be `low`, `moderate`, or `high`
   - `date` must be valid YYYY-MM-DD format and not in the future
2. Append a new row to the exercise table:
   ```
   | YYYY-MM-DD | type | duration_min | intensity | notes |
   ```
3. Write audit entry: `[HH:MM] exercise-log: logged {type} {duration}min ({intensity})`

### list

1. Parse `--range` into a start date (e.g., `7d` = 7 days ago from today)
2. Filter rows by date range
3. If `--type` is provided, further filter by type
4. Display matching rows in a formatted table
5. Show count: "Showing N workouts from {start} to {end}"

### streak

1. Extract all unique dates from the exercise log
2. Calculate current streak: count consecutive days ending at today (or most recent logged day) where at least one workout exists
3. Calculate longest streak: longest run of consecutive days with workouts
4. Display:
   - Current streak: N days
   - Longest streak: N days
   - Last 14 days calendar: use checkmarks and blanks to show activity

### summary

1. Filter entries by `--range` (default 7d)
2. If `--type` provided, filter by type
3. Calculate and display:
   - Total workouts in period
   - Total duration (hours and minutes)
   - Average workout duration
   - Breakdown by type (count and total duration per type)
   - Most active day of week
   - Intensity distribution (low/moderate/high counts)
   - Comparison to previous period of same length (if data available)

## Output

Format all output as clean markdown. Use tables for tabular data.

For `log`, confirm the entry:
```
Logged: 30min run (moderate) on 2026-02-28
Current streak: 5 days
```

For `list`, show the filtered table with a count header.

For `streak`, show streak stats and a visual calendar.

For `summary`, show a structured report with headers for each metric.
