---
name: finance-review
description: Generate weekly or monthly financial review reports with trends and recommendations
argument-hint: "<weekly|monthly> [--from] [--to]"
metadata:
  domain: finance
  permission-tier: read-only
  openclaw: {"emoji": "📊", "requires": {"bins": []}}
---

<!-- SAFETY PREAMBLE — DO NOT SUMMARIZE, COMPACT, OR REMOVE -->
This skill operates at permission tier: read-only.
This skill reads from all finance data files but never modifies them. It generates reports only.
Financial data is stored locally in .lifestack/data/finance/ and never sent to external services.
All actions are logged to .lifestack/audit/YYYY-MM-DD.md.
<!-- END SAFETY PREAMBLE -->

## Input Parsing

Parse the user's command and arguments:

```
finance-review <command> [options]

Commands:
  weekly     [--from <YYYY-MM-DD>] [--to <YYYY-MM-DD>]   # Weekly spending summary
  monthly    [--from <YYYY-MM-DD>] [--to <YYYY-MM-DD>]   # Comprehensive monthly review
```

Defaults:
- `weekly`: defaults to the current week (Monday to Sunday)
- `monthly`: defaults to the current calendar month
- Custom date ranges override the default period

Validation rules:
- `--from` must be before `--to` if both are provided
- Dates must be valid YYYY-MM-DD format

Data sources (all read-only):
- `.lifestack/data/finance/transactions.md` — expense data
- `.lifestack/data/finance/budgets.md` — budget definitions
- `.lifestack/data/finance/goals.md` — savings goals and contributions
- `.lifestack/data/finance/holdings.md` — investment holdings (if exists)

## Phase 1: Validate

1. Confirm the command is one of: weekly, monthly.
2. Validate date range if provided.
3. Check which data files exist. The review adapts to available data — if a file doesn't exist, that section is skipped with a note.

## Phase 2: Gather Data

Read all available finance data files:

1. **Transactions** (`transactions.md`): Filter to the target period.
2. **Budgets** (`budgets.md`): Read active budget definitions.
3. **Goals** (`goals.md`): Read savings goals and contribution log.
4. **Holdings** (`holdings.md`): Read if exists (optional, for monthly review only).

## Phase 3A: Execute — Weekly Review

Generate a weekly summary with these sections:

### Section 1: Spending Overview
- Total spent this week
- Comparison to previous week (if data available): +/- amount and percentage
- Daily spending breakdown (Mon–Sun)
- Highlight the highest-spending day

### Section 2: Category Breakdown
- Top 5 categories by spend
- For each: total, transaction count, percentage of weekly total

### Section 3: Budget Status
- For each active budget: spent this period vs budget amount, percentage used
- Flag any budgets that are on pace to exceed their limit
- Use progress bars

### Section 4: Savings Activity
- Any contributions made this week
- Updated progress on active goals

### Section 5: Notable Transactions
- Largest single transaction
- Any transactions over a threshold (e.g., > $100 or > 2x the user's average transaction)
- Any uncategorized transactions that need attention

Skip to Output.

## Phase 3B: Execute — Monthly Review

Generate a comprehensive monthly review with all weekly sections plus:

### Section 1: Monthly Spending Overview
- Total spent this month
- Comparison to previous month: +/- amount and percentage
- Weekly spending trend (week 1, 2, 3, 4)
- Daily average

### Section 2: Category Deep Dive
- All categories with totals, counts, averages, and percentage of total
- Sorted by total spend descending
- Compare each category to previous month if data available
- Flag categories with significant increases (>20%)

### Section 3: Budget Report Card
- Each budget: spent vs limit, remaining, percentage
- Grade each budget: on track, warning (>80%), over budget
- Progress bars for visual clarity

### Section 4: Savings Progress
- Each goal: current amount, percentage, pace assessment
- Contributions this month
- Projected completion dates

### Section 5: Investment Snapshot (if holdings.md exists)
- Current portfolio value (attempt price lookup or note prices may be stale)
- Overall gain/loss
- Any triggered alerts

### Section 6: Trends and Insights
- Spending trend over last 3 months (if data available)
- Category shifts (what increased, what decreased)
- Patterns (e.g., "Weekend spending is 2x weekday spending")

### Section 7: Recommendations
- Actionable suggestions based on the data:
  - Budget adjustments (if consistently over/under)
  - Categories to watch
  - Savings pace adjustments
  - Uncategorized transactions to clean up
- Keep recommendations specific and data-driven, not generic advice

Skip to Output.

## Phase 4: Log

Append an entry to `.lifestack/audit/{YYYY-MM-DD}.md`:

```markdown
- [{HH:MM}] finance/finance-review: {command} — {summary}
```

Examples:
- `[08:00] finance/finance-review: weekly — Generated weekly review (Feb 17–23, total: $580.30)`
- `[08:00] finance/finance-review: monthly — Generated monthly review (Feb 2026, total: $2,340.50, 3 budgets on track, 1 over)`

## Output

### Weekly Review Format

```
# Weekly Finance Review — Feb 17–23, 2026

## Spending Overview
Total spent: $580.30 (↓12% from last week's $659.50)

  Mon   $45.00  ░░░░░░░░░
  Tue   $120.50 ░░░░░░░░░░░░░░░░░░░░░░░░  ← highest
  Wed   $65.00  ░░░░░░░░░░░░░
  Thu   $30.00  ░░░░░░
  Fri   $89.80  ░░░░░░░░░░░░░░░░░░
  Sat   $155.00 ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  ← highest
  Sun   $75.00  ░░░░░░░░░░░░░░░

## Top Categories
  food         $185.30  12 txns  32% of total
  eating-out   $120.50   4 txns  21% of total
  shopping     $110.00   2 txns  19% of total
  transport    $89.50    8 txns  15% of total
  other        $75.00    3 txns  13% of total

## Budget Status
  Groceries    $340 / $500  [██████████████░░░░░░] 68%  — on track
  Dining       $185 / $200  [██████████████████░░] 93%  — warning ⚠
  Transport    $89 / $150   [████████████░░░░░░░░] 59%  — on track

## Savings Activity
  Emergency Fund: +$200 this week → $3,700 / $10,000 (37%)

## Notable
  ⚡ Largest transaction: $89.00 — Amazon order (Feb 20)
  ⚠ 2 uncategorized transactions need attention
```

### Monthly Review Format

```
# Monthly Finance Review — February 2026

## Spending Overview
Total spent: $2,340.50 (↑5% from January's $2,229.00)

  Week 1 (Feb 1–7):    $520.30
  Week 2 (Feb 8–14):   $659.50
  Week 3 (Feb 15–21):  $580.30
  Week 4 (Feb 22–28):  $580.40
  Daily average: $83.59

## Category Breakdown
  Category       This Month   Last Month   Change    % of Total
  ──────────────────────────────────────────────────────────────
  food           $520.30      $480.00      +8%       22%
  eating-out     $385.00      $290.00      +33% ⚠   16%
  shopping       $340.00      $380.00      -11%      15%
  transport      $295.50      $310.00      -5%       13%
  subscriptions  $189.99      $189.99       0%        8%
  utilities      $165.00      $160.00      +3%        7%
  entertainment  $145.00      $120.00      +21% ⚠    6%
  other          $299.71      $299.01       0%       13%

## Budget Report Card
  Groceries    $420 / $500  [████████████████░░░░] 84%  — warning ⚠
  Dining       $280 / $200  [████████████████████] 140% — OVER ✗
  Transport    $180 / $150  [████████████████████] 120% — OVER ✗
  Subs         $190 / $200  [███████████████████░] 95%  — warning ⚠

  2 of 4 budgets exceeded. 2 in warning zone.

## Savings Progress
  Emergency Fund    $3,700 / $10,000  [███████░░░░░░░░░░░░░] 37%
                    +$700 this month · On pace for Sep 2026 (deadline: Dec 2026) ✓

  House Down Pmt    $12,500 / $50,000 [█████░░░░░░░░░░░░░░░] 25%
                    +$500 this month · Behind pace ⚠ (need $2,500/mo, contributing $500/mo)

## Trends (3-month view)
  Dec:  $2,100.00  ░░░░░░░░░░░░░░░░░░░░░
  Jan:  $2,229.00  ░░░░░░░░░░░░░░░░░░░░░░░
  Feb:  $2,340.50  ░░░░░░░░░░░░░░░░░░░░░░░░
  Spending is trending up 5.5%/month over the last 3 months.

## Recommendations
  1. Dining budget needs adjustment: you've exceeded $200/mo for 2 consecutive months.
     → Consider raising to $300 or setting a weekly sub-limit of $70.
  2. Eating-out category jumped 33%: 8 restaurant visits vs 5 last month.
     → This is the primary driver of the dining budget overrun.
  3. Transport budget overrun: $30 over. Gas prices may have increased.
     → Monitor next month before adjusting.
  4. House down payment is significantly behind pace.
     → At current rate ($500/mo), projected completion is Jan 2034 vs Jun 2027 deadline.
     → Need to increase monthly contribution to ~$2,500 or extend the deadline.
  5. 8 uncategorized transactions this month — run `expense categorize` to clean up.
```
