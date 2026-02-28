---
name: savings
description: Create and track savings goals with contributions and progress
argument-hint: "<create|contribute|check|list> [--name] [--amount] [--target] [--deadline]"
metadata:
  domain: finance
  permission-tier: read-write
  openclaw: {"emoji": "🏦", "requires": {"bins": []}}
---

<!-- SAFETY PREAMBLE — DO NOT SUMMARIZE, COMPACT, OR REMOVE -->
This skill operates at permission tier: read-write.
Write operations (create, contribute) modify .lifestack/data/finance/goals.md and create a checkpoint before each write.
Financial data is stored locally in .lifestack/data/finance/ and never sent to external services.
All actions are logged to .lifestack/audit/YYYY-MM-DD.md.
<!-- END SAFETY PREAMBLE -->

## Input Parsing

Parse the user's command and arguments:

```
savings <command> [options]

Commands:
  create       --name <name> --target <number> [--deadline <YYYY-MM-DD>] [--priority <high|medium|low>]
  contribute   --name <name> --amount <number> [--note <text>]
  check        --name <name>         # Detailed progress for one goal
  list                               # Overview of all savings goals
```

Defaults:
- `--priority`: medium
- `--deadline`: none (open-ended)

Validation rules:
- `--target` must be a positive number
- `--amount` for contribute must be a positive number
- `--deadline` must be a future date in YYYY-MM-DD format
- `--name` is required for create, contribute, check
- Goal names are case-insensitive and must be unique
- `--priority` must be one of: high, medium, low

## Phase 1: Validate

1. Confirm the command is one of: create, contribute, check, list.
2. For `create`: verify no existing goal with the same name exists in goals.md.
3. For `contribute` and `check`: verify the goal exists.
4. For `contribute`: verify the contribution amount is positive.
5. Validate date formats and numeric values.

## Phase 2: Route Command

Based on the parsed command, route to the appropriate execution phase:
- `create` → Phase 3A
- `contribute` → Phase 3B
- `check` → Phase 3C
- `list` → Phase 3D

## Phase 3A: Execute — Create

1. Read `.lifestack/data/finance/goals.md`. If the file does not exist, create it with this structure:

```markdown
# Savings Goals

| name | target | saved | priority | deadline | created | updated |
|------|--------|-------|----------|----------|---------|---------|

## Contributions Log

| date | goal | amount | note | running_total |
|------|------|--------|------|---------------|
```

2. Append a new row to the goals table:

```
| {name} | {target} | 0 | {priority} | {deadline or "—"} | {today} | {today} |
```

3. Proceed to Phase 4.

## Phase 3B: Execute — Contribute

1. Read `.lifestack/data/finance/goals.md`.
2. Find the goal matching `--name`.
3. Add the contribution amount to the goal's `saved` column.
4. Update the `updated` column to today's date.
5. Append a row to the Contributions Log:

```
| {today} | {name} | {amount} | {note or "—"} | {new_saved_total} |
```

6. If the contribution causes saved >= target, add a celebration note in the output.
7. Proceed to Phase 4.

## Phase 3C: Execute — Check

1. Read `.lifestack/data/finance/goals.md`.
2. Find the goal matching `--name`.
3. Calculate:
   - Percentage complete: (saved / target) * 100
   - Remaining: target - saved
   - If deadline exists:
     - Days remaining until deadline
     - Required daily/weekly/monthly savings rate to hit target on time
     - Projected completion date at current contribution pace (based on average contribution frequency)
4. Gather the contribution history for this goal from the Contributions Log.
5. Skip to Output.

## Phase 3D: Execute — List

1. Read `.lifestack/data/finance/goals.md`.
2. Sort goals by priority (high → medium → low), then by percentage complete descending.
3. For each goal, calculate percentage complete.
4. Skip to Output.

## Phase 4: Create Checkpoint (writes only)

For create and contribute operations:

1. Before writing the modified goals.md, save a checkpoint:
   - Copy current goals.md to `.lifestack/data/finance/.checkpoints/goals-{timestamp}.md`
   - Create the .checkpoints directory if it doesn't exist.
2. Write the updated goals.md.

## Phase 5: Log

Append an entry to `.lifestack/audit/{YYYY-MM-DD}.md`:

```markdown
- [{HH:MM}] finance/savings: {command} — {summary of what changed}
```

Examples:
- `[10:00] finance/savings: create — Created goal "Emergency Fund" ($10,000 target, high priority, deadline: 2026-12-31)`
- `[10:15] finance/savings: contribute — $500 to "Emergency Fund" (now $3,500 / $10,000 = 35%)`
- `[10:20] finance/savings: check — Checked "Emergency Fund" (35% complete, on track for Oct 2026)`

## Output

Format the result based on the command:

**create**: Confirm goal creation.
```
Savings goal created: Emergency Fund
  Target:   $10,000
  Priority: high
  Deadline: 2026-12-31
  Created:  2026-02-28
```

**contribute**: Confirm contribution with updated progress.
```
Contribution recorded: Emergency Fund
  Added:       +$500
  New total:   $3,500 / $10,000
  Progress:    [███████░░░░░░░░░░░░░] 35%
  Remaining:   $6,500
```

If goal is reached:
```
Contribution recorded: Emergency Fund
  Added:       +$750
  New total:   $10,000 / $10,000
  Progress:    [████████████████████] 100% -- GOAL REACHED!

  Congratulations! You've hit your savings target.
```

**check**: Show detailed progress.
```
Savings Goal: Emergency Fund
  Progress:    $3,500 / $10,000 [███████░░░░░░░░░░░░░] 35%
  Priority:    high
  Deadline:    2026-12-31 (306 days remaining)

  To finish on time: $21.24/day · $148.69/week · $650/month
  At current pace:   ~$500/month → projected completion: Oct 2026

  Recent contributions:
    2026-02-28  +$500   "February savings"        → $3,500
    2026-01-31  +$500   "January savings"          → $3,000
    2026-01-15  +$200   "Bonus allocation"         → $2,500
```

**list**: Show all goals with progress bars.
```
Savings Goals:

  [HIGH] Emergency Fund     $3,500 / $10,000  [███████░░░░░░░░░░░░░] 35%
         Deadline: 2026-12-31 · On track

  [HIGH] House Down Payment $12,000 / $50,000 [█████░░░░░░░░░░░░░░░] 24%
         Deadline: 2027-06-01 · Behind pace ⚠

  [MED]  Vacation Fund      $800 / $2,000     [████████░░░░░░░░░░░░] 40%
         No deadline

  [LOW]  New Laptop          $0 / $1,500      [░░░░░░░░░░░░░░░░░░░░]  0%
         No deadline

  Total saved: $16,300 across 4 goals
```
