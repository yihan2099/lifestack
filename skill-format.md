# Skill Format Specification

This document defines the format for lifestack skills. Every `.md` file in the skill directories must follow this specification.

## File Structure

A lifestack skill file has four sections in order:

1. YAML frontmatter
2. Safety preamble
3. Description
4. Execution phases

```markdown
---
name: skill-name
description: One-line description
argument-hint: "<command> [--flags]"
metadata:
  domain: finance
  permission-tier: read-write
  openclaw: {"emoji": "...", "requires": {"bins": []}}
---

<!-- DO NOT SUMMARIZE/COMPACT/REMOVE -->
## Safety Preamble
...
<!-- END DO NOT SUMMARIZE/COMPACT/REMOVE -->

## Description
...

## Execution
### Phase 1: Input Parsing
...
### Phase 2: Validation
...
### Phase 3: Execution
...
### Phase 4: Output
...
```

## YAML Frontmatter

The frontmatter is enclosed in `---` delimiters and contains skill metadata in YAML format.

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Skill identifier. Lowercase, hyphen-separated. Must match the filename (e.g., `budget-track` for `budget-track.md`). |
| `description` | string | One-line description of what the skill does. Used in listings and search. |
| `argument-hint` | string | Usage hint showing the command and available flags. Displayed when users ask for help. |

### Metadata Fields

The `metadata` object contains classification and compatibility information.

| Field | Type | Values | Description |
|-------|------|--------|-------------|
| `domain` | string | `finance`, `social`, `health`, `core` | The domain this skill belongs to. Determines the data path. |
| `permission-tier` | string | `read-only`, `read-write`, `approval-required`, `destructive` | The safety tier. See SAFETY.md for enforcement rules. |
| `openclaw` | object | `{"emoji": "...", "requires": {"bins": [...]}}` | OpenClaw compatibility metadata. `emoji` is the skill's display icon. `requires.bins` lists any CLI tools the skill depends on. |

### Example Frontmatter

```yaml
---
name: budget-track
description: Track daily expenses against monthly budget categories
argument-hint: "<amount> <category> [--note 'description']"
metadata:
  domain: finance
  permission-tier: read-write
  openclaw: {"emoji": "💰", "requires": {"bins": []}}
---
```

## Safety Preamble

Every skill must include a safety preamble immediately after the frontmatter. The preamble is wrapped in context-integrity markers:

```markdown
<!-- DO NOT SUMMARIZE/COMPACT/REMOVE -->
## Safety Preamble

**Permission tier: read-write**

This skill can read and write files in `.lifestack/data/finance/`. It cannot:
- Modify files outside `.lifestack/data/finance/`
- Make network requests or external API calls
- Delete existing data

Before any write operation:
1. Create a checkpoint via the checkpoint skill
2. Write only to `.lifestack/data/finance/`
3. Log the action to `.lifestack/audit/YYYY-MM-DD.md`
<!-- END DO NOT SUMMARIZE/COMPACT/REMOVE -->
```

### Preamble Requirements

- The `<!-- DO NOT SUMMARIZE/COMPACT/REMOVE -->` markers are mandatory. They instruct agents to preserve this section during context compaction.
- State the permission tier explicitly.
- List what the skill can and cannot do.
- Specify the pre-action requirements (checkpoint, audit logging).
- Tailor the constraints to the specific permission tier. A `read-only` skill's preamble looks different from an `approval-required` skill's preamble.

## Description

A plain-language description of what the skill does, when to use it, and what output to expect. Written for the agent that will execute it.

Keep it concise. The agent needs to understand the skill's purpose and behavior, not read a tutorial.

## Execution Phases

Skills follow a 4-phase execution pattern. Each phase is a `### Phase N:` subsection under `## Execution`.

### Phase 1: Input Parsing

Parse the user's command and arguments. Define:
- Required arguments and their types
- Optional flags and defaults
- Argument validation rules (e.g., amount must be positive, category must exist)

```markdown
### Phase 1: Input Parsing

Parse the command arguments:
- `<amount>` (required): Positive number. The expense amount.
- `<category>` (required): String. Must match an existing budget category in `.lifestack/data/finance/budget.md`.
- `--note` (optional): String. Description of the expense. Default: empty.

If `<amount>` is not a positive number, respond with an error and do not proceed.
If `<category>` does not exist, list available categories and ask the user to choose.
```

### Phase 2: Validation

Verify preconditions before making changes. This phase catches errors before any side effects occur.

```markdown
### Phase 2: Validation

1. Verify `.lifestack/data/finance/budget.md` exists. If not, instruct the user to run the `onboard` skill first.
2. Verify the category exists in the budget file.
3. Check that adding this expense would not exceed any configured alerts (e.g., >90% of category budget).
```

### Phase 3: Execution

The core logic. All writes, external actions, and state changes happen here. This phase must:
- Create a checkpoint before any write (for `read-write` and above)
- Perform the operation
- Log to the audit trail

```markdown
### Phase 3: Execution

1. Create a checkpoint of `.lifestack/data/finance/expenses-YYYY-MM.md`.
2. Append a new entry to `.lifestack/data/finance/expenses-YYYY-MM.md`:
   ```
   | YYYY-MM-DD | <amount> | <category> | <note> |
   ```
3. Log to `.lifestack/audit/YYYY-MM-DD.md`:
   - Skill: budget-track
   - Command: `<amount> <category> [--note]`
   - Status: success
   - Checkpoint: <checkpoint-id>
```

### Phase 4: Output

What to display to the user after execution. Keep output focused and actionable.

```markdown
### Phase 4: Output

Display:
- Confirmation: "Logged $<amount> under <category>."
- Running total: "$X spent of $Y budget for <category> this month."
- If over 80% of budget: warn "You've used <percent>% of your <category> budget."
```

## Permission Enforcement

Each permission tier has specific enforcement requirements that the skill's execution phases must respect.

### read-only

- Phase 3 must not contain any file write, create, or delete operations.
- Phase 3 must not contain any network requests.
- Audit logging is still required (the audit log is the one exception to the no-write rule for read-only skills).

### read-write

- Phase 3 must create a checkpoint before any write.
- Writes are restricted to `.lifestack/data/{domain}/`.
- No file deletions. Use append or overwrite semantics only.
- No network requests.

### approval-required

- Phase 3 must include a preview step showing the user exactly what will happen.
- Phase 3 must pause for explicit user confirmation before the external action.
- Both the preview and the user's response must be logged in the audit trail.

### destructive

- Phase 3 must include a preview step with an explicit "This action is irreversible" warning.
- Phase 3 must pause for two separate user confirmations.
- A final checkpoint must be created before execution.

## Data Path Conventions

Skills read and write data in domain-specific directories:

| Domain | Data Path |
|--------|-----------|
| `finance` | `.lifestack/data/finance/` |
| `social` | `.lifestack/data/social/` |
| `health` | `.lifestack/data/health/` |
| `core` | `.lifestack/data/` (root) |

File naming conventions:
- Time-series data: `{name}-YYYY-MM.md` (e.g., `expenses-2026-02.md`)
- Configuration: `{name}.yaml` (e.g., `budget.yaml`)
- Reports: `{name}-report-YYYY-MM-DD.md`

All data files are markdown or YAML. No binary formats, no databases, no JSON.

## Audit Logging

Every skill execution must produce an audit log entry, regardless of outcome. See SAFETY.md for the full audit trail specification.

The log entry must include:
- Timestamp (HH:MM:SS)
- Skill name
- Full command with arguments
- Permission tier
- Summary of inputs
- Summary of outputs or error
- Status (success, failure, declined, aborted)
- Checkpoint ID (if a checkpoint was created)

## Example: Complete Skill Template

```markdown
---
name: example-skill
description: A template showing the complete skill format
argument-hint: "<arg1> [--flag value]"
metadata:
  domain: core
  permission-tier: read-write
  openclaw: {"emoji": "📋", "requires": {"bins": []}}
---

<!-- DO NOT SUMMARIZE/COMPACT/REMOVE -->
## Safety Preamble

**Permission tier: read-write**

This skill can read and write files in `.lifestack/data/`. It cannot:
- Modify files outside `.lifestack/data/`
- Make network requests
- Delete existing data

Before any write: create a checkpoint, then log to the audit trail.
<!-- END DO NOT SUMMARIZE/COMPACT/REMOVE -->

## Description

This is a template skill demonstrating the standard format. Use it as a starting point when creating new skills.

## Execution

### Phase 1: Input Parsing

Parse the command arguments:
- `<arg1>` (required): Description and validation rules.
- `--flag` (optional): Description. Default: default_value.

### Phase 2: Validation

1. Verify preconditions.
2. Check that required files exist.
3. Validate argument values against expected ranges/formats.

### Phase 3: Execution

1. Create a checkpoint of affected files.
2. Perform the operation.
3. Log to `.lifestack/audit/YYYY-MM-DD.md`.

### Phase 4: Output

Display results to the user. Include confirmation, summary, and any warnings.
```
