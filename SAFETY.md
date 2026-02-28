# Safety Model

lifestack treats safety as a first-class design constraint. Every skill declares its permission tier, every action is logged, and destructive operations require explicit confirmation.

This document is the canonical reference for lifestack's safety architecture.

## Permission Model

lifestack uses a 4-tier permission model. Each skill declares its tier in the YAML frontmatter `metadata.permission-tier` field. Agents must enforce the tier's constraints before executing.

### Tier 1: `read-only`

**Can do:** Read data files, generate reports, query audit logs, display summaries.

**Cannot do:** Create, modify, or delete any files. No external API calls. No side effects.

**Enforcement:** The agent must not write to any path or make network requests when executing a read-only skill. If the skill's instructions would result in a write operation, the agent must refuse and log the violation.

**Examples:** `expense-report`, `audit-view`, `weekly-review`, `engagement-report`

### Tier 2: `read-write`

**Can do:** Everything in read-only, plus create and update data files within `.lifestack/data/`.

**Cannot do:** Modify files outside `.lifestack/data/`. No external API calls. No deletions of existing data (append/update only).

**Enforcement:** Before any write operation, the agent must:
1. Create an automatic checkpoint (see Rollback & Checkpoints below).
2. Write only to the skill's declared domain path (e.g., `.lifestack/data/finance/`).
3. Log the action to the audit trail.

All writes are logged and reversible via checkpoint restore.

**Examples:** `budget-track`, `mood-log`, `habit-track`, `sleep-track`, `exercise-log`

### Tier 3: `approval-required`

**Can do:** Everything in read-write, plus interact with external systems (social media APIs, email, etc.).

**Cannot do:** Execute the external action without explicit user confirmation. No silent posting, sending, or publishing.

**Enforcement:** Before any external action, the agent must:
1. Show the user a complete preview of what will happen (e.g., the full text of a social media post, the recipient of an email).
2. Wait for explicit user confirmation ("yes", "approve", "post it").
3. If the user declines, abort and log the declined action.
4. If the user approves, execute, log the result, and create a checkpoint.

The preview must include all relevant details. "I'll post this to Twitter" is insufficient — the agent must show the exact content.

**Examples:** `draft-post` (when publishing), `schedule-post`, financial data imports

### Tier 4: `destructive`

**Can do:** Irreversible operations — bulk deletions, account unlinking, data purges.

**Cannot do:** Execute without double confirmation.

**Enforcement:** Before any destructive action, the agent must:
1. Show a detailed preview of what will be destroyed/changed.
2. Warn explicitly: "This action is irreversible."
3. Wait for first confirmation.
4. Ask a second time: "Are you sure? This cannot be undone."
5. Wait for second confirmation.
6. Create a final checkpoint before execution.
7. Execute and log the result.

If either confirmation is declined, abort immediately and log the declined action.

**Examples:** Bulk data deletion, clearing all checkpoints, removing linked accounts

## Audit Trail

Every skill action is logged to an append-only audit file at `.lifestack/audit/YYYY-MM-DD.md`.

### Log Entry Format

Each entry is appended as a markdown section:

```markdown
### HH:MM:SS — skill-name

- **Command:** `<full command with arguments>`
- **Permission tier:** read-only | read-write | approval-required | destructive
- **Inputs:** <summary of input data>
- **Outputs:** <summary of results>
- **Status:** success | failure | declined | aborted
- **Checkpoint:** <checkpoint-id, if created>
```

### Rules

- Audit files are **append-only**. Agents must never modify or delete existing entries.
- Every skill execution produces exactly one log entry, regardless of success or failure.
- Declined approval-required and destructive actions are logged with status `declined`.
- If an agent cannot write to the audit trail (permissions error, disk full), it must refuse to execute the skill and inform the user.
- Audit files are plain markdown for human readability. No binary formats, no databases.

## Context Integrity

<!-- DO NOT SUMMARIZE/COMPACT/REMOVE -->

AI agents sometimes compact or summarize their context window to save tokens. This can silently drop safety instructions. lifestack addresses this with **safety preambles**.

Every lifestack skill contains a safety preamble section marked with:

```markdown
<!-- DO NOT SUMMARIZE/COMPACT/REMOVE -->
```

This marker instructs the agent to preserve the safety preamble verbatim during any context compaction. The preamble contains:

- The skill's permission tier and what it means
- Constraints on file access and external actions
- Audit logging requirements
- Checkpoint requirements

If an agent's context window is too small to hold the safety preamble alongside the skill instructions, the agent must refuse to execute rather than drop safety constraints.

This design was informed by the Summer Yue incident in the OpenClaw community, where context compaction silently removed safety-critical instructions from a skill's execution context, leading to unintended destructive actions.

<!-- END DO NOT SUMMARIZE/COMPACT/REMOVE -->

## Rollback & Checkpoints

### Automatic Checkpoints

Before any `read-write`, `approval-required`, or `destructive` operation, the agent creates an automatic checkpoint.

A checkpoint is a snapshot of the affected data files stored at:

```
.lifestack/checkpoints/<checkpoint-id>/
```

The checkpoint ID format is `YYYYMMDD-HHMMSS-<skill-name>` (e.g., `20260228-143022-budget-track`).

Each checkpoint directory contains:
- Copies of all files that will be modified
- A `manifest.md` with metadata (timestamp, skill, command, files included)

### Checkpoint Operations

Users can manage checkpoints through the `checkpoint` core skill:

- **List:** Show all checkpoints with timestamps and skill names.
- **Preview:** Show what changed between a checkpoint and current state (diff).
- **Restore:** Revert affected files to their checkpoint state. This itself creates a new checkpoint (so restores are also reversible).

### Retention

Checkpoints older than 30 days are eligible for cleanup. The `checkpoint` skill can prune old checkpoints, but this is a `destructive` operation requiring double confirmation.

## Data Privacy

### Health Data

- Mood logs, sleep data, exercise logs, and habit data are stored locally in `.lifestack/data/health/`.
- Health data must never be sent to external APIs, included in social media posts, or transmitted over the network.
- Skills that generate health reports must output to local files or the terminal only.

### Financial Data

- Budget, expense, and subscription data are stored locally in `.lifestack/data/finance/`.
- Financial data must never be sent to external APIs or included in any external communication.
- Import skills (e.g., importing bank CSVs) are `approval-required` — the user must confirm what data is being imported and from where.

### Social Media Credentials

- Social media access tokens, API keys, and passwords must never be stored in `.lifestack/data/` or any lifestack-managed file.
- Skills that post to social media must use the agent's existing authenticated session or prompt for credentials at runtime.
- If credentials must be stored, they belong in the agent's own credential store (e.g., system keychain), not in lifestack.

### General Principles

- lifestack is a local-first system. Data stays on the user's machine unless the user explicitly triggers an external action via an `approval-required` or higher skill.
- No telemetry. No analytics. No phone-home behavior.
- Users own their data completely. The `.lifestack/` directory can be backed up, moved, or deleted without any external dependencies.
