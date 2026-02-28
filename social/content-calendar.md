---
name: content-calendar
description: Plan, schedule, and manage social media content across platforms
argument-hint: "<plan|add|list|next> [--platform twitter|linkedin|substack|all] [--period week|month] [--status draft|scheduled|posted]"
metadata:
  domain: social
  permission-tier: read-write
  openclaw: {"emoji": "📅", "requires": {"bins": []}}
---

<!-- SAFETY PREAMBLE — DO NOT SUMMARIZE, COMPACT, OR REMOVE -->
This skill operates at permission tier: read-write.
Read-write skills may create and modify data files in .lifestack/data/social/.
No data is deleted without explicit user confirmation.
Social media credentials are NEVER stored in lifestack data files.
All actions are logged to .lifestack/audit/YYYY-MM-DD.md.
<!-- END SAFETY PREAMBLE -->

## Input Parsing

Parse the user's command and arguments:

| Token | Meaning | Default |
|-------|---------|---------|
| `plan` | Generate a content plan for a period | — |
| `add` | Add a specific post to the calendar | — |
| `list` | Show calendar entries with filters | — |
| `next` | Show next upcoming scheduled item | — |
| `--platform <p>` | Target platform (twitter, linkedin, substack, all) | all |
| `--period <p>` | Time period (week, month) | week |
| `--status <s>` | Filter by status (draft, scheduled, posted) | all |
| `--topic <t>` | Topic or theme for the content | — |
| `--date <d>` | Specific date (YYYY-MM-DD) | today |

If no command is provided, default to `list --period week`.

Validate:
- Command must be one of: plan, add, list, next
- Platform must be one of: twitter, linkedin, substack, all
- Period must be one of: week, month
- Status must be one of: draft, scheduled, posted
- Date must be valid YYYY-MM-DD format

## Phase 1: Load Calendar Data

Read the calendar data file at `.lifestack/data/social/calendar.md`.

If the file does not exist, create it with this header:

```markdown
# Social Content Calendar

| date | platform | topic | status | notes |
|------|----------|-------|--------|-------|
```

Parse the markdown table into structured data for processing.

## Phase 2: Execute Command

### plan

Generate a content plan based on inputs:

1. Determine the date range from `--period` (week = 7 days from today, month = 30 days from today)
2. Review existing calendar entries to avoid conflicts
3. Generate a balanced plan considering:
   - Platform-specific posting frequency (Twitter: 3-5/week, LinkedIn: 2-3/week, Substack: 1/week)
   - Topic variety — alternate between content types
   - Optimal posting days per platform
4. Present the plan as a table for user review
5. On user confirmation, append entries to calendar with status `draft`

### add

Add a single entry to the calendar:

1. Require: date, platform, topic
2. Optional: status (default: draft), notes
3. Validate the date is not in the past
4. Check for conflicts (same platform, same date already has an entry)
5. If conflict exists, warn user and ask for confirmation
6. Append the new row to the calendar table

### list

Display calendar entries with filters:

1. Apply filters: `--platform`, `--period`, `--status`
2. Sort by date ascending
3. Display as a formatted markdown table
4. Include summary counts: total items, by status, by platform

### next

Show the next upcoming item:

1. Filter entries with status `scheduled` or `draft`
2. Find the entry with the earliest date >= today
3. Display the entry with full details
4. If no upcoming items, suggest running `plan`

## Phase 3: Persist Changes

For `plan` and `add` commands:

1. Write updated calendar data back to `.lifestack/data/social/calendar.md`
2. Maintain the markdown table format
3. Keep entries sorted by date ascending

## Output

### plan
```
Content plan for {period} ({start_date} to {end_date}):

| date | platform | topic | status | notes |
|------|----------|-------|--------|-------|
| ...  | ...      | ...   | draft  | ...   |

{count} posts planned across {platforms}.
Add to calendar? (yes/no)
```

### add
```
Added to content calendar:
  Date: {date}
  Platform: {platform}
  Topic: {topic}
  Status: {status}
```

### list
```
Content calendar ({period}):

| date | platform | topic | status | notes |
|------|----------|-------|--------|-------|
| ...  | ...      | ...   | ...    | ...   |

Summary: {total} items — {draft} draft, {scheduled} scheduled, {posted} posted
```

### next
```
Next scheduled content:
  Date: {date}
  Platform: {platform}
  Topic: {topic}
  Status: {status}
  Notes: {notes}
```

Log the action to `.lifestack/audit/YYYY-MM-DD.md`:
```
- HH:MM — social/content-calendar {command} — {summary of action}
```
