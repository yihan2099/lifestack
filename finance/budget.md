---
name: budget
description: Create, track, and manage spending budgets by category and period
argument-hint: "<create|update|check|list|delete> [--name] [--amount] [--period] [--categories]"
metadata:
  domain: finance
  permission-tier: read-write
  openclaw: {"emoji": "💰", "requires": {"bins": []}}
---

<!-- SAFETY PREAMBLE — DO NOT SUMMARIZE, COMPACT, OR REMOVE -->
This skill operates at permission tier: read-write.
Write operations (create, update) modify .lifestack/data/finance/budgets.md and create a checkpoint before each write.
Delete operations require explicit user confirmation before execution (approval-required override).
Financial data is stored locally in .lifestack/data/finance/ and never sent to external services.
All actions are logged to .lifestack/audit/YYYY-MM-DD.md.
<!-- END SAFETY PREAMBLE -->

## Input Parsing

Parse the user's command and arguments:

```
budget <command> [options]

Commands:
  create   --name <name> --amount <number> --period <weekly|monthly|yearly> [--categories <cat1,cat2,...>]
  update   --name <name> [--amount <number>] [--period <weekly|monthly|yearly>] [--categories <cat1,cat2,...>]
  check    [--name <name>]          # Check spending vs budget; omit --name for all budgets
  list                              # List all active budgets
  delete   --name <name>            # Remove a budget (requires confirmation)
```

Defaults:
- `--period`: monthly
- `--categories`: "general" if not specified

Validation rules:
- `--amount` must be a positive number
- `--period` must be one of: weekly, monthly, yearly
- `--name` is required for create, update, delete
- Budget names are case-insensitive and must be unique

## Phase 1: Validate

1. Confirm the command is one of: create, update, check, list, delete.
2. For `create`: verify no existing budget with the same name exists in budgets.md.
3. For `update` and `delete`: verify the budget exists.
4. For `delete`: prompt the user for explicit confirmation before proceeding. Do NOT delete without confirmation. Say: "This will permanently delete the budget '{name}'. Type 'yes' to confirm."
5. Validate amount is a positive number if provided.
6. Validate period is one of the allowed values.

## Phase 2: Route Command

Based on the parsed command, route to the appropriate execution phase:
- `create` → Phase 3A
- `update` → Phase 3B
- `check` → Phase 3C
- `list` → Phase 3D
- `delete` → Phase 3E

## Phase 3A: Execute — Create

1. Read `.lifestack/data/finance/budgets.md`. If the file does not exist, create it with this header:

```markdown
# Budgets

| name | amount | period | categories | created | updated |
|------|--------|--------|------------|---------|---------|
```

2. Append a new row:

```
| {name} | {amount} | {period} | {categories} | {today} | {today} |
```

3. Proceed to Phase 4.

## Phase 3B: Execute — Update

1. Read `.lifestack/data/finance/budgets.md`.
2. Find the row matching `--name` (case-insensitive).
3. Update only the fields that were provided (amount, period, categories).
4. Set the `updated` column to today's date.
5. Proceed to Phase 4.

## Phase 3C: Execute — Check

1. Read `.lifestack/data/finance/budgets.md` to get budget definitions.
2. Read `.lifestack/data/finance/transactions.md` to get spending data.
3. For the target budget (or all budgets if `--name` is omitted):
   a. Determine the current period window (e.g., for monthly: first day of current month to today).
   b. Sum transactions matching the budget's categories within the period window.
   c. Calculate: spent, remaining, percentage used.
4. Skip to Output — display results with progress bars.

Progress bar format:
```
Groceries (monthly): $340 / $500 [████████████░░░░░░░░] 68%
                     $160 remaining · 12 days left in period
```

If over budget:
```
Dining (monthly):    $280 / $200 [████████████████████] 140% ⚠ OVER
                     $80 over budget · 12 days left in period
```

## Phase 3D: Execute — List

1. Read `.lifestack/data/finance/budgets.md`.
2. Display all budgets in a clean table format.
3. Skip to Output.

## Phase 3E: Execute — Delete

1. This phase only runs AFTER user confirmation was received in Phase 1.
2. Read `.lifestack/data/finance/budgets.md`.
3. Remove the row matching `--name`.
4. Proceed to Phase 4.

## Phase 4: Create Checkpoint (writes only)

For create, update, and delete operations:

1. Before writing the modified budgets.md, save a checkpoint:
   - Copy current budgets.md to `.lifestack/data/finance/.checkpoints/budgets-{timestamp}.md`
   - Create the .checkpoints directory if it doesn't exist.
2. Write the updated budgets.md.

## Phase 5: Log

Append an entry to `.lifestack/audit/{YYYY-MM-DD}.md`:

```markdown
- [{HH:MM}] finance/budget: {command} — {summary of what changed}
```

Examples:
- `[14:32] finance/budget: create — Created budget "Groceries" ($500/monthly, categories: food, groceries)`
- `[14:35] finance/budget: check — Checked 3 budgets (1 over budget: Dining at 140%)`
- `[14:40] finance/budget: delete — Deleted budget "Old Subscriptions"`

## Output

Format the result based on the command:

**create**: Confirm creation with all details.
```
Budget created: Groceries
  Amount:     $500 / monthly
  Categories: food, groceries
  Created:    2026-02-28
```

**update**: Show what changed (before → after).
```
Budget updated: Groceries
  Amount:     $400 → $500
  Categories: food → food, groceries
```

**check**: Show progress bars (see Phase 3C format).

**list**: Show all budgets in a table.
```
Active Budgets:
  Groceries    $500/mo   food, groceries        68% used
  Dining       $200/mo   restaurants, delivery  140% ⚠ OVER
  Transport    $150/mo   gas, transit            22% used
```

**delete**: Confirm deletion.
```
Budget deleted: Old Subscriptions
  Checkpoint saved: .checkpoints/budgets-20260228-1440.md
```
