---
name: audit-trail
description: Append-only logging and viewing of all lifestack skill actions
argument-hint: "log <entry>" or "view [--date YYYY-MM-DD] [--skill name] [--last N]"
metadata:
  domain: core
  permission-tier: read-write
  openclaw: {"emoji": "📋", "requires": {"bins": []}}
---

<!-- SAFETY PREAMBLE — DO NOT SUMMARIZE, COMPACT, OR REMOVE -->
This skill operates at permission tier: read-write.
Read-write skills may create and modify files in the .lifestack/ directory.
Audit logs are append-only — existing entries must never be modified or deleted.
All actions are logged to .lifestack/audit/YYYY-MM-DD.md.
<!-- END SAFETY PREAMBLE -->

## Input Parsing

Parse the command into one of two modes:

**Log mode**: `audit-trail log <entry>`
- `entry`: A string describing the action to log. May include structured fields separated by `|`.
- The entry is appended to today's audit file.

**View mode**: `audit-trail view [--date YYYY-MM-DD] [--skill name] [--last N]`
- `--date`: Show entries from a specific date (defaults to today)
- `--skill`: Filter entries to only show those from a specific skill
- `--last N`: Show the last N entries across all dates (overrides `--date`)
- Flags can be combined: `--skill checkpoint --last 10` shows last 10 checkpoint entries

Validate:
- In log mode, `entry` must not be empty
- In view mode, `--date` must be valid YYYY-MM-DD format if provided
- In view mode, `--last N` must be a positive integer if provided

## Phase 1: Route

Determine which operation to execute:
- If first argument is `log` -> Log mode
- If first argument is `view` -> View mode
- Otherwise -> Return error with usage hint

## Phase 2: Execute — Log Mode

1. **Determine file path**: `.lifestack/audit/{YYYY-MM-DD}.md` using today's date
2. **Create directory** if `.lifestack/audit/` doesn't exist
3. **Create file** if today's audit file doesn't exist, with header:
   ```markdown
   # Audit Log — {YYYY-MM-DD}

   | Time | Skill | Command | Tier | Inputs | Result | Duration |
   |------|-------|---------|------|--------|--------|----------|
   ```
4. **Parse the entry**: Extract structured fields if provided in `skill|command|tier|inputs|result|duration` format. If entry is a plain string, set:
   - Skill: "unknown"
   - Command: the entry text
   - Tier: "n/a"
   - Inputs: "n/a"
   - Result: "logged"
   - Duration: "n/a"
5. **Append the row** to the markdown table:
   ```
   | {HH:MM:SS} | {skill} | {command} | {tier} | {inputs} | {result} | {duration} |
   ```
6. **Verify** the append by reading back the last line

CRITICAL: Audit logs are append-only. Never overwrite, truncate, or modify existing entries. Only append new rows to the table.

## Phase 3: Execute — View Mode

1. **Determine scope**:
   - If `--last N`: Scan audit files in reverse chronological order, collect entries until N is reached
   - If `--date`: Read only the specified date's file
   - If neither: Read today's file

2. **Read audit file(s)**: Parse the markdown table rows from each file

3. **Apply filters**:
   - If `--skill` is set: Keep only rows where the Skill column matches
   - If `--last N` is set: Keep only the last N entries after filtering

4. **Handle empty results**: If no entries match, display a message indicating no matching entries were found

## Phase 4: Format Output

**For log mode**, return:

```
audit-trail logged

file:  .lifestack/audit/{YYYY-MM-DD}.md
time:  {HH:MM:SS}
entry: {formatted entry summary}
```

**For view mode**, return the matching entries as a formatted table:

```
audit-trail view ({count} entries)

| Time     | Skill      | Command              | Tier        | Result  |
|----------|------------|----------------------|-------------|---------|
| 09:15:32 | checkpoint | create "before edit" | read-write  | success |
| 09:15:33 | onboard    | --domain finance     | read-write  | success |
| ...      | ...        | ...                  | ...         | ...     |
```

The view output omits the Inputs and Duration columns for readability. Include them only if the user requests verbose output.

## Output

Return the formatted result as shown above.

- Log mode: Confirmation of the appended entry with file path and timestamp
- View mode: Filtered and formatted table of audit entries with count
