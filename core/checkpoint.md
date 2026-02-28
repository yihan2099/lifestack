---
name: checkpoint
description: Creates state snapshots for safe rollback of lifestack file changes
argument-hint: "create <description>" or "auto <file-path>" or "list [--last N]" or "clean [--older-than 30d]"
metadata:
  domain: core
  permission-tier: read-write
  openclaw: {"emoji": "💾", "requires": {"bins": []}}
---

<!-- SAFETY PREAMBLE — DO NOT SUMMARIZE, COMPACT, OR REMOVE -->
This skill operates at permission tier: read-write.
Read-write skills may create and modify files in the .lifestack/ directory.
Checkpoints capture file state before modifications to enable safe rollback.
All actions are logged to .lifestack/audit/YYYY-MM-DD.md.
<!-- END SAFETY PREAMBLE -->

## Input Parsing

Parse the command into one of four modes:

**Create mode**: `checkpoint create <description>`
- `description`: Required. A human-readable description of why this checkpoint is being created (e.g., "Before updating finance goals").
- Captures the current state of all files in `.lifestack/` (excluding `audit/` and `checkpoints/` directories).

**Auto mode**: `checkpoint auto <file-path>`
- `file-path`: Required. The specific file about to be modified.
- Called programmatically by other skills before write operations.
- Only captures the state of the specified file, not the entire `.lifestack/` directory.

**List mode**: `checkpoint list [--last N]`
- `--last N`: Optional. Show the last N checkpoints (default: 10).

**Clean mode**: `checkpoint clean [--older-than 30d]`
- `--older-than`: Optional. Duration threshold for cleaning old checkpoints (default: 30d). Accepts formats like `7d`, `30d`, `90d`.

Validate:
- In create mode, `description` must not be empty
- In auto mode, `file-path` must point to an existing file
- In clean mode, duration must be a valid format (positive integer followed by `d`)

## Phase 1: Route

Determine which operation to execute:
- If first argument is `create` -> Create mode
- If first argument is `auto` -> Auto mode
- If first argument is `list` -> List mode
- If first argument is `clean` -> Clean mode
- Otherwise -> Return error with usage hint

## Phase 2: Execute — Create Mode

1. **Generate checkpoint ID**: `cp-{short-hash}` where short-hash is derived from timestamp + random component (8 characters, alphanumeric)
2. **Create directory** if `.lifestack/checkpoints/` doesn't exist
3. **Collect file states**: Read the current contents of all files in `.lifestack/` except:
   - `.lifestack/audit/` — audit logs are append-only and not rolled back
   - `.lifestack/checkpoints/` — avoid recursive checkpoint-of-checkpoints
4. **Write checkpoint file** at `.lifestack/checkpoints/{checkpoint-id}.md`:

```markdown
# Checkpoint {checkpoint-id}

- **ID**: {checkpoint-id}
- **Timestamp**: {YYYY-MM-DD HH:MM:SS}
- **Description**: {description}
- **Type**: manual
- **Affected Files**: {count}

## Files

### {relative-file-path-1}
\`\`\`
{full file contents}
\`\`\`

### {relative-file-path-2}
\`\`\`
{full file contents}
\`\`\`
```

5. **Log via audit-trail**: `audit-trail log "checkpoint|create {checkpoint-id}|read-write|{description}|success|{duration}"`

## Phase 3: Execute — Auto Mode

1. **Generate checkpoint ID**: `cp-{short-hash}` (same scheme as create mode)
2. **Create directory** if `.lifestack/checkpoints/` doesn't exist
3. **Read the specific file**: Capture the current contents of the file at `file-path`
4. **Write checkpoint file** at `.lifestack/checkpoints/{checkpoint-id}.md`:

```markdown
# Checkpoint {checkpoint-id}

- **ID**: {checkpoint-id}
- **Timestamp**: {YYYY-MM-DD HH:MM:SS}
- **Description**: Auto: {file-path}
- **Type**: auto
- **Affected Files**: 1

## Files

### {relative-file-path}
\`\`\`
{full file contents}
\`\`\`
```

5. **Log via audit-trail**: `audit-trail log "checkpoint|auto {file-path}|read-write|{file-path}|success|{duration}"`

NOTE: Auto mode is designed to be fast and lightweight. It captures only the single file about to be modified, making it suitable for programmatic use by other skills.

## Phase 4: Execute — List Mode

1. **Scan checkpoints**: Read all `.md` files in `.lifestack/checkpoints/`
2. **Parse each checkpoint**: Extract ID, timestamp, description, type, affected file count from the frontmatter section
3. **Sort by timestamp** descending (most recent first)
4. **Apply limit**: Show last N entries (default: 10)
5. **Format as table**:

```
checkpoint list ({total} total, showing {displayed})

| ID         | Timestamp           | Type   | Description                | Files |
|------------|---------------------|--------|----------------------------|-------|
| cp-a1b2c3d4| 2026-02-28 09:15:32 | manual | Before updating finance    | 5     |
| cp-e5f6g7h8| 2026-02-28 09:15:30 | auto   | Auto: .lifestack/config.md | 1     |
| ...        | ...                 | ...    | ...                        | ...   |
```

## Phase 5: Execute — Clean Mode

1. **Parse threshold**: Convert `--older-than` value to a date cutoff (e.g., `30d` = 30 days before today)
2. **Scan checkpoints**: Read all checkpoint files and parse their timestamps
3. **Identify candidates**: Find checkpoints older than the cutoff date
4. **Display candidates**:
   ```
   checkpoint clean — {count} checkpoint(s) older than {threshold}

   | ID         | Timestamp           | Description                |
   |------------|---------------------|----------------------------|
   | cp-x1y2z3  | 2026-01-15 08:00:00 | Before profile update      |
   | ...        | ...                 | ...                        |

   Remove these checkpoints? (yes/no)
   ```
5. **If approved**: Delete the checkpoint files and log each deletion
6. **If denied**: Cancel and log the cancellation
7. **Log via audit-trail**: `audit-trail log "checkpoint|clean --older-than {threshold}|read-write|{count} candidates|{deleted count} removed|{duration}"`

## Phase 6: Log

All operations are logged in their respective execution phases above. This phase is a verification step:
- Confirm that the audit-trail log entry was written
- If audit-trail logging failed, write a warning but do not fail the checkpoint operation (checkpoints are critical safety infrastructure)

## Output

**Create mode**:
```
checkpoint created

id:          {checkpoint-id}
type:        manual
description: {description}
files:       {count} captured
path:        .lifestack/checkpoints/{checkpoint-id}.md
```

**Auto mode**:
```
checkpoint created

id:          {checkpoint-id}
type:        auto
file:        {file-path}
path:        .lifestack/checkpoints/{checkpoint-id}.md
```

**List mode**: Table output as shown in Phase 4.

**Clean mode**:
```
checkpoint clean complete

threshold: {--older-than value}
reviewed:  {candidate count}
removed:   {deleted count}
retained:  {kept count}
```
