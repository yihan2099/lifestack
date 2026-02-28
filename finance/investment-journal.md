---
name: investment-journal
description: Record investment theses, conviction updates, and post-mortems for positions
argument-hint: "<thesis|update|review|postmortem|list> [--asset] [--conviction]"
metadata:
  domain: finance
  permission-tier: read-write
  openclaw: {"emoji": "📓", "requires": {"bins": []}}
---

<!-- SAFETY PREAMBLE — DO NOT SUMMARIZE, COMPACT, OR REMOVE -->
This skill operates at permission tier: read-write.
Write operations (thesis, update, postmortem) modify .lifestack/data/finance/journal.md and create a checkpoint before each write.
Investment theses may contain sensitive financial strategy. This data never leaves the local machine.
Financial data is stored locally in .lifestack/data/finance/ and never sent to external services.
All actions are logged to .lifestack/audit/YYYY-MM-DD.md.
<!-- END SAFETY PREAMBLE -->

## Input Parsing

Parse the user's command and arguments:

```
investment-journal <command> [options]

Commands:
  thesis      --asset <ticker|name> --why <text> [--bull <text>] [--bear <text>] [--target <number>] [--horizon <text>] [--conviction <1-5>]
  update      --asset <ticker|name> [--conviction <1-5>] [--notes <text>]
  review      --asset <ticker|name>              # Review thesis vs reality
  postmortem  --asset <ticker|name> --right <text> --wrong <text> --different <text>
  list        [--active] [--closed] [--sort <conviction|date|asset>]
```

Defaults:
- `--conviction`: 3 (moderate)
- `--horizon`: "medium-term (6-12 months)"
- `--sort` for list: conviction (highest first)
- `list` defaults to `--active` (only active theses)

Conviction scale:
- 1 — Very low: speculative, small position
- 2 — Low: interesting but uncertain
- 3 — Moderate: reasonable thesis, standard position
- 4 — High: strong conviction, willing to add on dips
- 5 — Very high: highest conviction, core position

Validation rules:
- `--asset` is required for thesis, update, review, postmortem
- `--why` is required for thesis (core rationale)
- `--conviction` must be an integer 1-5
- `--right`, `--wrong`, `--different` are all required for postmortem
- For `update`: at least one of `--conviction` or `--notes` must be provided
- For `postmortem`: the thesis must exist; it will be marked as closed

## Phase 1: Validate

1. Confirm the command is one of: thesis, update, review, postmortem, list.
2. For `thesis`: verify no active thesis already exists for this asset. If one exists, suggest using `update` instead. Say: "An active thesis for {asset} already exists (created {date}). Use `investment-journal update --asset {asset}` to update it, or close it with `postmortem` first."
3. For `update`, `review`, `postmortem`: verify an active thesis exists for this asset. If not, say: "No active thesis found for {asset}. Use `investment-journal thesis --asset {asset}` to create one."
4. Validate conviction range (1-5).

## Phase 2: Route Command

Based on the parsed command, route to the appropriate execution phase:
- `thesis` → Phase 3A
- `update` → Phase 3B
- `review` → Phase 3C
- `postmortem` → Phase 3D
- `list` → Phase 3E

## Phase 3A: Execute — Thesis

1. Read `.lifestack/data/finance/journal.md`. If the file does not exist, create it with this structure:

```markdown
# Investment Journal

## Active Theses

<!-- Each thesis is a section with structured fields. Do not reorder fields. -->

## Closed Positions

<!-- Post-mortems for closed positions are appended here. -->

## Conviction History

| date | asset | old_conviction | new_conviction | notes |
|------|-------|----------------|----------------|-------|
```

2. Append a new thesis entry under "## Active Theses":

```markdown
### {ASSET} — Opened {date}

- **Status**: Active
- **Conviction**: {conviction}/5
- **Why**: {why}
- **Bull case**: {bull or "Not specified"}
- **Bear case**: {bear or "Not specified"}
- **Target price**: {target or "Not specified"}
- **Time horizon**: {horizon}
- **Entry date**: {date}
- **Last updated**: {date}
```

3. Log the initial conviction in the Conviction History table:

```
| {date} | {asset} | — | {conviction} | Initial thesis |
```

4. Proceed to Phase 4.

## Phase 3B: Execute — Update

1. Read `.lifestack/data/finance/journal.md`.
2. Find the active thesis for the specified asset.
3. If `--conviction` is provided:
   a. Read the current conviction value.
   b. Update the conviction field in the thesis entry.
   c. Append a row to the Conviction History table:
   ```
   | {date} | {asset} | {old_conviction} | {new_conviction} | {notes or "Conviction update"} |
   ```
4. If `--notes` is provided:
   a. Append a timestamped note below the thesis entry's fields:
   ```markdown
   - **{date}**: {notes}
   ```
5. Update the "Last updated" field to today's date.
6. Proceed to Phase 4.

## Phase 3C: Execute — Review

1. Read `.lifestack/data/finance/journal.md`.
2. Find the active thesis for the specified asset.
3. Gather supporting data:
   a. Read `.lifestack/data/finance/portfolio.md` (if exists) for transaction history and cost basis.
   b. Read `.lifestack/data/finance/holdings.md` (if exists) for current price alerts.
   c. Attempt to look up current price via web search.
4. Compile the review:
   - Original thesis summary (why, bull case, bear case)
   - Current conviction and conviction history (all changes over time)
   - Current price vs entry price and target price
   - Time elapsed vs stated time horizon
   - Any notes/updates that have been added
5. Generate assessment questions (do NOT answer them — prompt the user to reflect):
   - "Is the original thesis still intact?"
   - "Has anything changed that invalidates the bull or bear case?"
   - "Would you open this position today at the current price?"
   - "Should conviction be updated?"
6. Skip to Output.

## Phase 3D: Execute — Postmortem

1. Read `.lifestack/data/finance/journal.md`.
2. Find the active thesis for the specified asset.
3. Move the thesis from "## Active Theses" to "## Closed Positions".
4. Update the thesis entry:
   - Change **Status** from "Active" to "Closed — {date}"
   - Append postmortem fields:
   ```markdown
   - **Closed**: {date}
   - **What went right**: {right}
   - **What went wrong**: {wrong}
   - **What I'd do differently**: {different}
   ```
5. If portfolio data exists, calculate and include:
   - Total return on the position (realized gain/loss)
   - Holding period
6. Log final conviction change to 0 in Conviction History:
   ```
   | {date} | {asset} | {old_conviction} | 0 | Position closed — postmortem recorded |
   ```
7. Proceed to Phase 4.

## Phase 3E: Execute — List

1. Read `.lifestack/data/finance/journal.md`.
2. Based on flags:
   - `--active` (default): show only active theses
   - `--closed`: show only closed positions
   - Both flags: show all
3. Sort by the specified field:
   - `conviction`: highest conviction first (default)
   - `date`: most recently created/updated first
   - `asset`: alphabetical
4. For each thesis, extract: asset, conviction, entry date, time horizon, last updated.
5. Skip to Output.

## Phase 4: Create Checkpoint (writes only)

For thesis, update, and postmortem operations:

1. Before writing the modified journal.md, save a checkpoint:
   - Copy current journal.md to `.lifestack/data/finance/.checkpoints/journal-{timestamp}.md`
   - Create the .checkpoints directory if it doesn't exist.
2. Write the updated journal.md.

## Phase 5: Log

Append an entry to `.lifestack/audit/{YYYY-MM-DD}.md`:

```markdown
- [{HH:MM}] finance/investment-journal: {command} — {summary}
```

Examples:
- `[10:00] finance/investment-journal: thesis — Created thesis for AAPL (conviction: 4/5, target: $220, horizon: 6-12 months)`
- `[10:15] finance/investment-journal: update — Updated AAPL conviction 4 → 3 ("Earnings miss, thesis weakened")`
- `[10:20] finance/investment-journal: review — Reviewed AAPL thesis (held 4 months, conviction: 3/5, current: $185 vs target: $220)`
- `[10:30] finance/investment-journal: postmortem — Closed AAPL position (held 8 months, +12.3% return, lessons recorded)`
- `[10:35] finance/investment-journal: list — Listed 5 active theses (highest conviction: BTC at 5/5)`

## Output

Format the result based on the command:

**thesis**: Confirm the new thesis.
```
Investment thesis recorded: AAPL

  Conviction:   ████░ 4/5 — High
  Why:          Strong services revenue growth, AI integration potential
  Bull case:    Services hits $100B ARR, Apple Intelligence drives upgrade cycle
  Bear case:    China slowdown, regulatory pressure on App Store fees
  Target price: $220
  Time horizon: 6-12 months
  Entry date:   2026-02-28
```

**update**: Show what changed.
```
Thesis updated: AAPL

  Conviction:   ████░ 4/5 → ███░░ 3/5  (↓ decreased)
  Notes added:  "Earnings miss in Q1, services growth slowing. Thesis weakened but not broken."
  Last updated: 2026-02-28

  Conviction history:
    2026-02-28  4 → 3  Earnings miss, thesis weakened
    2025-10-15  — → 4  Initial thesis
```

**review**: Show thesis vs reality.
```
Thesis Review: AAPL

  ── Original Thesis ──────────────────────────────────────────
  Why:          Strong services revenue growth, AI integration potential
  Bull case:    Services hits $100B ARR, Apple Intelligence drives upgrade cycle
  Bear case:    China slowdown, regulatory pressure on App Store fees
  Target:       $220
  Horizon:      6-12 months

  ── Current State ────────────────────────────────────────────
  Conviction:   ███░░ 3/5  (was 4/5 at entry)
  Entry price:  $185.50  (2025-10-15)
  Current:      $192.00  (+3.5%)
  Target:       $220.00  (still 14.6% away)
  Time held:    4 months, 13 days (within 6-12 month horizon)

  ── Conviction History ───────────────────────────────────────
    2026-02-28  4 → 3  Earnings miss, thesis weakened
    2025-10-15  — → 4  Initial thesis

  ── Updates ──────────────────────────────────────────────────
    2026-02-28: Earnings miss in Q1, services growth slowing.

  ── Reflection Questions ─────────────────────────────────────
    • Is the original thesis still intact?
    • Has anything changed that invalidates the bull or bear case?
    • Would you open this position today at $192.00?
    • Should conviction be updated?
```

**postmortem**: Show the closed position summary.
```
Position closed: AAPL — Postmortem recorded

  ── Summary ──────────────────────────────────────────────────
  Held:            2025-10-15 → 2026-06-15 (8 months)
  Return:          +12.3% ($185.50 → $208.30)
  Final conviction: 3/5

  ── What went right ──────────────────────────────────────────
  Services revenue did accelerate in Q2-Q3. AI features drove upgrade cycle
  as predicted.

  ── What went wrong ──────────────────────────────────────────
  China revenue declined more than expected. Didn't anticipate the regulatory
  fine in the EU.

  ── What I'd do differently ──────────────────────────────────
  Would have set a tighter stop-loss given the China exposure. Should have
  reduced position size when conviction dropped to 3.

  Thesis moved to Closed Positions in journal.
```

**list**: Show all theses in a summary view.
```
Active Investment Theses

  Asset     Conviction   Entry Date    Horizon        Last Updated
  ──────────────────────────────────────────────────────────────────
  BTC       █████ 5/5    2025-06-01    1-2 years      2026-02-15
  NVDA      ████░ 4/5    2025-09-20    6-12 months    2026-01-10
  AAPL      ███░░ 3/5    2025-10-15    6-12 months    2026-02-28
  SOL       ██░░░ 2/5    2026-01-05    3-6 months     2026-02-01
  ARKK      █░░░░ 1/5    2026-02-20    speculative    2026-02-20
  ──────────────────────────────────────────────────────────────────
  5 active theses · Avg conviction: 3.0

  Tip: Run `investment-journal review --asset <ticker>` for a deep dive.
```

Closed positions list (with `--closed`):
```
Closed Positions

  Asset     Conviction   Held              Return      Closed
  ──────────────────────────────────────────────────────────────
  TSLA      3/5 (peak 4) 2025-03 → 2025-08 +18.2%     2025-08-15
  META      4/5 (peak 4) 2025-01 → 2025-06 -5.1%      2025-06-20
  ──────────────────────────────────────────────────────────────
  2 closed positions · Avg return: +6.5%

  Tip: Run `investment-journal review --asset <ticker>` for the full postmortem.
```
