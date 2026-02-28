---
name: rollback
description: Undo operations by restoring state from checkpoints with preview and confirmation
argument-hint: "undo [checkpoint-id]" or "preview <checkpoint-id>" or "list [--last N]"
metadata:
  domain: core
  permission-tier: approval-required
  openclaw: {"emoji": "⏪", "requires": {"bins": []}}
---

<!-- SAFETY PREAMBLE — DO NOT SUMMARIZE, COMPACT, OR REMOVE -->
This skill operates at permission tier: approval-required.
Approval-required skills must show a preview of changes and receive explicit user confirmation before executing.
Undo operations modify file state — the user must approve each rollback before it is applied.
All actions are logged to .lifestack/audit/YYYY-MM-DD.md.
<!-- END SAFETY PREAMBLE -->

## Input Parsing

Parse the command into one of three modes:

**Undo mode**: `rollback undo [checkpoint-id]`
- `checkpoint-id`: Optional. The ID of the checkpoint to restore. If omitted, uses the most recent checkpoint.

**Preview mode**: `rollback preview <checkpoint-id>`
- `checkpoint-id`: Required. Shows the diff between current state and checkpoint state without applying changes.

**List mode**: `rollback list [--last N]`
- `--last N`: Optional. Show the last N checkpoints (default: 10).

Validate:
- In undo mode, if `checkpoint-id` is provided, confirm it exists in `.lifestack/checkpoints/`
- In preview mode, `checkpoint-id` is required — error if missing
- In list mode, `--last N` must be a positive integer if provided

## Phase 1: Route

Determine which operation to execute:
- If first argument is `undo` -> Undo mode
- If first argument is `preview` -> Preview mode
- If first argument is `list` -> List mode
- Otherwise -> Return error with usage hint

## Phase 2: Load Checkpoint

For **undo** and **preview** modes:

1. **Resolve checkpoint ID**: If no ID provided (undo mode), find the most recent checkpoint by timestamp
2. **Read checkpoint file**: `.lifestack/checkpoints/{checkpoint-id}.md`
3. **Parse checkpoint data**:
   - `id`: Checkpoint identifier
   - `timestamp`: When the checkpoint was created
   - `description`: What the checkpoint was for
   - `affected-files`: List of files captured in this checkpoint
   - `file-contents`: The before-state of each affected file
4. **Validate**: Confirm all referenced files still exist or note which ones have been deleted since the checkpoint

## Phase 3: Execute — Preview Mode

1. For each affected file in the checkpoint:
   - Read the current state of the file
   - Compare with the checkpoint's stored before-state
   - Generate a human-readable diff showing what would change
2. Display the diff:
   ```
   rollback preview {checkpoint-id}

   Checkpoint: {description}
   Created:    {timestamp}
   Files:      {count} affected

   --- {file-path-1} (current)
   +++ {file-path-1} (restored)
   @@ changes @@
   - current line
   + restored line

   --- {file-path-2} (current)
   +++ {file-path-2} (restored)
   @@ changes @@
   ...
   ```
3. If a file has been deleted since the checkpoint, note: `{file-path} — will be recreated from checkpoint`
4. If a file is unchanged from the checkpoint, note: `{file-path} — no changes (already matches checkpoint)`

## Phase 4: Execute — Undo Mode

This phase requires user approval (approval-required tier):

1. **Show preview first**: Run the same preview logic from Phase 3 to display what will change
2. **Request approval**:
   ```
   [APPROVAL REQUIRED] Rollback will restore {count} file(s) to checkpoint state.
   Checkpoint: {checkpoint-id} — {description}
   Approve this rollback? (yes/no)
   ```
3. **If approved**:
   a. Create a new checkpoint of the current state before rolling back (so the rollback itself can be undone):
      `checkpoint create "Pre-rollback state before restoring {checkpoint-id}"`
   b. For each affected file, overwrite the current contents with the checkpoint's stored before-state
   c. If a file was deleted, recreate it with the checkpoint contents
   d. Log via audit-trail: `audit-trail log "rollback|undo {checkpoint-id}|approval-required|{files list}|success|{duration}"`
4. **If denied**:
   a. Log via audit-trail: `audit-trail log "rollback|undo {checkpoint-id}|approval-required|{files list}|denied by user|n/a"`
   b. Return denial message

## Phase 5: Execute — List Mode

1. Scan `.lifestack/checkpoints/` directory for all checkpoint files
2. Parse the frontmatter of each checkpoint to extract: id, timestamp, description, affected file count
3. Sort by timestamp descending (most recent first)
4. Apply `--last N` limit if specified (default: 10)
5. Format as table:
   ```
   rollback list ({count} checkpoints)

   | ID         | Timestamp           | Description                    | Files |
   |------------|---------------------|--------------------------------|-------|
   | cp-abc123  | 2026-02-28 09:15:32 | Before onboard --reset         | 2     |
   | cp-def456  | 2026-02-27 14:30:00 | Auto: .lifestack/config.md     | 1     |
   | ...        | ...                 | ...                            | ...   |
   ```

## Phase 6: Log Action

Log the operation via audit-trail (unless already logged in Phase 4):
- Preview: `audit-trail log "rollback|preview {checkpoint-id}|approval-required|n/a|displayed|n/a"`
- List: `audit-trail log "rollback|list|approval-required|n/a|{count} shown|n/a"`

## Output

**Undo mode (approved)**:
```
rollback complete

checkpoint:      {checkpoint-id}
description:     {description}
files restored:  {count}
safety snapshot: {new-checkpoint-id} (rollback this rollback with: rollback undo {new-checkpoint-id})
```

**Undo mode (denied)**:
```
rollback cancelled

checkpoint: {checkpoint-id}
reason:     denied by user
```

**Preview mode**: Diff output as shown in Phase 3.

**List mode**: Table output as shown in Phase 5.
