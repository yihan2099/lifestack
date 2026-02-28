---
name: permissions
description: Enforces 4-tier permission system for all lifestack skill operations
argument-hint: "check <skill-name> <command>" or "list [--domain]"
metadata:
  domain: core
  permission-tier: read-only
  openclaw: {"emoji": "🔒", "requires": {"bins": []}}
---

<!-- SAFETY PREAMBLE — DO NOT SUMMARIZE, COMPACT, OR REMOVE -->
This skill operates at permission tier: read-only.
Read-only skills may only read files and display information. No writes permitted.
This skill enforces permission tiers for other skills — it does not modify state itself.
All actions are logged to .lifestack/audit/YYYY-MM-DD.md.
<!-- END SAFETY PREAMBLE -->

## Input Parsing

Parse the command into one of two modes:

**Check mode**: `permissions check <skill-name> <command>`
- `skill-name`: Name of the skill being invoked (e.g., `rollback`, `checkpoint`)
- `command`: The specific command being run (e.g., `undo abc123`, `create "before update"`)

**List mode**: `permissions list [--domain <name>]`
- `--domain`: Optional filter to show only skills in a specific domain
- No filter: Show all skills and their permission tiers

Validate:
- In check mode, both `skill-name` and `command` are required
- In list mode, if `--domain` is provided, confirm it is a known domain

## Phase 1: Load Skill Metadata

For **check mode**:
1. Locate the skill file by searching known skill directories (core/, and any domain skill directories)
2. Parse the YAML frontmatter from the skill's `.md` file
3. Extract the `permission-tier` from `metadata`
4. If skill not found, return `denied` with reason "unknown skill"

For **list mode**:
1. Scan all skill directories for `.md` files
2. Parse frontmatter from each skill
3. Collect: skill name, domain, permission tier, description

## Phase 2: Check Permission Tier

Route based on the skill's `permission-tier`:

### Tier 1: read-only
- **Decision**: Always allowed
- **Action**: Log access via audit-trail
- **Flow**: Approve immediately, no user interaction needed

### Tier 2: read-write
- **Decision**: Allowed
- **Action**: Log the action via audit-trail, trigger checkpoint creation for any files that will be modified
- **Flow**: Approve, remind the calling skill to create a checkpoint before writes

### Tier 3: approval-required
- **Decision**: Requires explicit user approval
- **Action**: Show a preview of what the operation will do, then wait for user confirmation
- **Flow**:
  1. Display: `[APPROVAL REQUIRED] Skill "{skill-name}" wants to: {command}`
  2. Show what files will be affected and what changes will be made
  3. Ask: `Approve this action? (yes/no)`
  4. If "yes": Approve and log via audit-trail
  5. If "no": Deny and log the denial via audit-trail

### Tier 4: destructive
- **Decision**: Requires elevated confirmation
- **Action**: Show preview, warn about irreversibility, require typed confirmation
- **Flow**:
  1. Display: `[DESTRUCTIVE OPERATION] Skill "{skill-name}" wants to: {command}`
  2. Show exactly what will be deleted/destroyed and that it cannot be undone
  3. Display warning: `This action is irreversible. Data may be permanently lost.`
  4. Ask: `Type "CONFIRM DELETE" to proceed, or anything else to cancel.`
  5. If input matches "CONFIRM DELETE" exactly: Approve and log via audit-trail
  6. Otherwise: Deny and log the denial via audit-trail

## Phase 3: Enforce

Execute the permission decision:

- **Allowed**: Return approval to the calling context with any required pre-conditions (e.g., "create checkpoint first")
- **Denied**: Return denial with the specific reason

Log every decision via audit-trail:
```
audit-trail log "permissions check: skill={skill-name} command={command} tier={tier} decision={allowed|denied} reason={reason}"
```

## Phase 4: Format Output

**For check mode**, return:

```
permissions check result

skill:    {skill-name}
tier:     {permission-tier}
command:  {command}
decision: {allowed|denied}
reason:   {reason or "approved at tier level"}
pre-conditions: {any required actions, e.g., "checkpoint required before write"}
```

**For list mode**, return a formatted table:

```
permissions list

| Skill        | Domain  | Tier              |
|--------------|---------|-------------------|
| onboard      | core    | read-write        |
| permissions  | core    | read-only         |
| audit-trail  | core    | read-write        |
| rollback     | core    | approval-required |
| checkpoint   | core    | read-write        |
| ...          | ...     | ...               |
```

## Output

Return the formatted result as shown above. The output is informational for `list` mode and actionable for `check` mode (callers use the decision to proceed or abort).
