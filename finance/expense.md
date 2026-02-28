---
name: expense
description: Log, list, summarize, categorize, and import expense transactions
argument-hint: "<log|list|summary|categorize|import> [--amount] [--category] [--date] [--file]"
metadata:
  domain: finance
  permission-tier: read-write
  openclaw: {"emoji": "💸", "requires": {"bins": []}}
---

<!-- SAFETY PREAMBLE — DO NOT SUMMARIZE, COMPACT, OR REMOVE -->
This skill operates at permission tier: read-write.
Write operations (log, categorize) modify .lifestack/data/finance/transactions.md and create a checkpoint before each write.
Import operations are elevated to approval-required because they perform bulk writes. A preview of all changes is shown before any data is written.
Financial data is stored locally in .lifestack/data/finance/ and never sent to external services.
All actions are logged to .lifestack/audit/YYYY-MM-DD.md.
<!-- END SAFETY PREAMBLE -->

## Input Parsing

Parse the user's command and arguments:

```
expense <command> [options]

Commands:
  log          --amount <number> --category <cat> [--description <text>] [--date <YYYY-MM-DD>] [--source <text>]
  list         [--from <YYYY-MM-DD>] [--to <YYYY-MM-DD>] [--category <cat>] [--min <number>] [--max <number>]
  summary      [--period <week|month|year>] [--from <YYYY-MM-DD>] [--to <YYYY-MM-DD>]
  categorize   [--dry-run]          # Auto-categorize uncategorized expenses
  import       --file <path>        # Import from CSV/text (approval-required)
```

Defaults:
- `--date`: today's date
- `--source`: "manual"
- `--period` for summary: month
- `--from` / `--to` for list: current month

Validation rules:
- `--amount` must be a positive number
- `--date` must be valid YYYY-MM-DD format
- `--category` should be a single word or hyphenated phrase (e.g., "food", "eating-out")

## Phase 1: Validate

1. Confirm the command is one of: log, list, summary, categorize, import.
2. For `log`: verify amount and category are provided.
3. For `import`: verify the file path exists and is readable. Confirm the file looks like CSV or structured text.
4. For `import`: **STOP and show a preview** of what will be imported. Do NOT write any data until the user explicitly approves. Say: "This will import {N} transactions. Here's a preview of the first 5:" followed by the preview table. Then ask: "Proceed with import? (yes/no)"
5. Validate date formats and numeric ranges.

## Phase 2: Route Command

Based on the parsed command, route to the appropriate execution phase:
- `log` → Phase 3A
- `list` → Phase 3B
- `summary` → Phase 3C
- `categorize` → Phase 3D
- `import` → Phase 3E

## Phase 3A: Execute — Log

1. Read `.lifestack/data/finance/transactions.md`. If the file does not exist, create it with this header:

```markdown
# Transactions

| date | amount | category | description | source |
|------|--------|----------|-------------|--------|
```

2. Append a new row:

```
| {date} | {amount} | {category} | {description} | {source} |
```

3. Proceed to Phase 4.

## Phase 3B: Execute — List

1. Read `.lifestack/data/finance/transactions.md`.
2. Filter rows based on provided criteria:
   - Date range: `--from` to `--to`
   - Category: exact match (case-insensitive)
   - Amount range: `--min` to `--max`
3. Sort by date descending (most recent first).
4. Skip to Output.

## Phase 3C: Execute — Summary

1. Read `.lifestack/data/finance/transactions.md`.
2. Determine the period window based on `--period` or `--from`/`--to`.
3. Group transactions by category.
4. Calculate per-category: total, count, average transaction size.
5. Calculate overall: grand total, daily average.
6. Rank categories by total spend (highest first).
7. Skip to Output.

## Phase 3D: Execute — Categorize

1. Read `.lifestack/data/finance/transactions.md`.
2. Find all rows where category is empty, "uncategorized", or "?".
3. For each uncategorized transaction, infer a category from the description using these heuristics:
   - Keywords mapping: "grocery/food/walmart/costco" → food, "uber/lyft/gas/parking" → transport, "netflix/spotify/subscription" → subscriptions, "restaurant/dining/coffee" → eating-out, "amazon/online" → shopping
   - If no keyword matches, mark as "other".
4. If `--dry-run`: show proposed categorizations without writing.
5. Otherwise: update the transactions in place.
6. Proceed to Phase 4 (if not dry-run).

## Phase 3E: Execute — Import

1. This phase only runs AFTER user approval of the preview in Phase 1.
2. Parse the import file:
   - CSV: expect columns for date, amount, category (optional), description (optional)
   - Handle common date formats and normalize to YYYY-MM-DD
   - Handle currency symbols (strip $, €, etc.)
3. For each parsed row, append to transactions.md with source = "import:{filename}".
4. Proceed to Phase 4.

## Phase 4: Create Checkpoint (writes only)

For log, categorize (non-dry-run), and import operations:

1. Before writing the modified transactions.md, save a checkpoint:
   - Copy current transactions.md to `.lifestack/data/finance/.checkpoints/transactions-{timestamp}.md`
   - Create the .checkpoints directory if it doesn't exist.
2. Write the updated transactions.md.

## Phase 5: Log

Append an entry to `.lifestack/audit/{YYYY-MM-DD}.md`:

```markdown
- [{HH:MM}] finance/expense: {command} — {summary of what changed}
```

Examples:
- `[09:15] finance/expense: log — Logged $45.20 in food ("Grocery run at Costco")`
- `[09:20] finance/expense: summary — Generated monthly summary (total: $2,340 across 8 categories)`
- `[09:25] finance/expense: categorize — Auto-categorized 12 transactions (food: 5, transport: 3, other: 4)`
- `[09:30] finance/expense: import — Imported 47 transactions from bank-export.csv`

## Output

Format the result based on the command:

**log**: Confirm the logged transaction.
```
Expense logged:
  Date:        2026-02-28
  Amount:      $45.20
  Category:    food
  Description: Grocery run at Costco
  Source:      manual
```

**list**: Show filtered transactions in a table.
```
Transactions (Feb 1–28, 2026):
  2026-02-28  $45.20   food         Grocery run at Costco
  2026-02-27  $12.50   eating-out   Coffee with Sarah
  2026-02-25  $89.00   shopping     Amazon order
  ...
  Showing 15 of 42 transactions · Total: $1,230.50
```

**summary**: Show category breakdown.
```
Spending Summary — February 2026

  Category       Total     Count   Avg
  ─────────────────────────────────────
  food           $420.30   12      $35.03
  eating-out     $285.00   8       $35.63
  transport      $180.50   15      $12.03
  shopping       $156.00   3       $52.00
  subscriptions  $89.99    4       $22.50
  ─────────────────────────────────────
  Total          $1,131.79 42      $26.95
  Daily avg      $40.42
```

**categorize**: Show what was categorized.
```
Auto-categorized 12 transactions:
  food:        5 transactions ($215.30)
  transport:   3 transactions ($45.00)
  eating-out:  2 transactions ($67.50)
  other:       2 transactions ($89.00)

  Checkpoint saved before changes.
```

**import**: Show import results.
```
Imported 47 transactions from bank-export.csv
  Date range:  2026-01-01 to 2026-01-31
  Total value: $3,450.22
  Categories:  12 categorized, 35 uncategorized
  Source tag:  import:bank-export.csv

  Tip: Run `expense categorize` to auto-categorize the 35 uncategorized transactions.
```
