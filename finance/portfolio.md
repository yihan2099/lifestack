---
name: portfolio
description: Log investment transactions and track holdings, performance, and allocation (record-keeping only)
argument-hint: "<log|holdings|performance|history|allocation> [--asset] [--type] [--from] [--to]"
metadata:
  domain: finance
  permission-tier: read-write
  openclaw: {"emoji": "💰", "requires": {"bins": []}}
---

<!-- SAFETY PREAMBLE — DO NOT SUMMARIZE, COMPACT, OR REMOVE -->
This skill operates at permission tier: read-write.
Write operations (log) modify .lifestack/data/finance/portfolio.md and create a checkpoint before each write.
This skill records investment transactions for tracking purposes. It NEVER executes trades, connects to brokerages, or moves money. All data is entered manually by the user or imported from records.
Financial data is stored locally in .lifestack/data/finance/ and never sent to external services.
All actions are logged to .lifestack/audit/YYYY-MM-DD.md.
<!-- END SAFETY PREAMBLE -->

## Input Parsing

Parse the user's command and arguments:

```
portfolio <command> [options]

Commands:
  log          --type <buy|sell> --asset <ticker|name> --quantity <number> --price <number> [--asset-class <class>] [--fees <number>] [--date <YYYY-MM-DD>] [--notes <text>]
  holdings     [--asset <ticker|name>]                   # Current holdings summary
  performance  [--asset <ticker|name>]                   # P&L per holding (requires current prices)
  history      [--asset <ticker|name>] [--type <buy|sell>] [--from <YYYY-MM-DD>] [--to <YYYY-MM-DD>]
  allocation   [--targets]                               # Allocation breakdown with optional target comparison
```

Defaults:
- `--date`: today's date
- `--fees`: 0
- `--asset-class`: inferred from asset if possible, otherwise "other"
- `--from` / `--to` for history: all time (no filter)

Asset class values: stocks, crypto, bonds, etfs, real-estate, commodities, cash, other

Validation rules:
- `--type` for log must be one of: buy, sell
- `--quantity` must be a positive number
- `--price` must be a positive number (price per unit at time of transaction)
- `--fees` must be a non-negative number
- `--date` must be valid YYYY-MM-DD format
- For `sell` transactions: verify the user holds enough of the asset (quantity check against current holdings)

## Phase 1: Validate

1. Confirm the command is one of: log, holdings, performance, history, allocation.
2. For `log`:
   a. Verify type, asset, quantity, and price are provided.
   b. If type is `sell`, read current holdings and verify sufficient quantity exists. If not, warn the user: "You currently hold {N} units of {asset}. Cannot sell {requested} units." and stop.
   c. Auto-calculate total cost: `(quantity * price) + fees` for buys, `(quantity * price) - fees` for sells.
3. Validate date formats and numeric values.

**CRITICAL**: This skill is a record-keeping tool only. If the user asks to "execute a trade", "place an order", "connect to a brokerage", or anything involving live trading, politely refuse and explain: "Portfolio is a record-keeping tool. It logs transactions that have already happened. It does not execute trades or connect to any brokerage."

## Phase 2: Route Command

Based on the parsed command, route to the appropriate execution phase:
- `log` → Phase 3A
- `holdings` → Phase 3B
- `performance` → Phase 3C
- `history` → Phase 3D
- `allocation` → Phase 3E

## Phase 3A: Execute — Log

1. Read `.lifestack/data/finance/portfolio.md`. If the file does not exist, create it with this structure:

```markdown
# Portfolio Transactions

| date | type | asset | asset_class | quantity | price | fees | total | notes |
|------|------|-------|-------------|----------|-------|------|-------|-------|

## Allocation Targets

| asset_class | target_pct |
|-------------|------------|
| stocks | 60 |
| bonds | 20 |
| crypto | 10 |
| cash | 10 |
```

2. Calculate total:
   - Buy: `total = (quantity * price) + fees`
   - Sell: `total = (quantity * price) - fees`

3. Append a new row to the transactions table:

```
| {date} | {type} | {asset} | {asset_class} | {quantity} | {price} | {fees} | {total} | {notes or "—"} |
```

4. Proceed to Phase 4.

## Phase 3B: Execute — Holdings

1. Read `.lifestack/data/finance/portfolio.md`.
2. Aggregate transactions per asset:
   - For each asset, sum buy quantities and subtract sell quantities to get current quantity.
   - Calculate average cost basis: total cost of all buys / total quantity bought (weighted average).
   - Exclude assets with zero remaining quantity (fully sold).
3. If `--asset` is provided, filter to that single asset.
4. Calculate allocation percentages:
   - Per-asset allocation: (asset_total_cost / portfolio_total_cost) * 100
5. Sort by total cost basis descending.
6. Skip to Output.

## Phase 3C: Execute — Performance

1. Read `.lifestack/data/finance/portfolio.md`.
2. Get current holdings (same aggregation as Phase 3B).
3. For each holding, determine current price:
   a. Check if `.lifestack/data/finance/holdings.md` exists (from invest-watch) and use its data.
   b. Use web search to look up current prices (search for "{asset} price today").
   c. If neither is available, ask the user to provide current prices.
4. Calculate per-holding:
   - Current value: current_quantity * current_price
   - Cost basis total: current_quantity * avg_cost_basis
   - Unrealized gain/loss: current_value - cost_basis_total
   - Unrealized gain/loss percentage: (unrealized_gain_loss / cost_basis_total) * 100
5. Calculate realized gains from sell transactions:
   - For each sell: realized = (sell_price - avg_cost_basis_at_time) * quantity - fees
6. Calculate portfolio totals:
   - Total current value, total cost basis, total unrealized gain/loss
   - Total realized gains from all closed positions
   - Overall return: (total_unrealized + total_realized) / total_cost_basis * 100
7. Skip to Output.

## Phase 3D: Execute — History

1. Read `.lifestack/data/finance/portfolio.md`.
2. Filter transactions based on provided criteria:
   - `--asset`: match asset name or ticker (case-insensitive)
   - `--type`: filter by buy or sell
   - `--from` / `--to`: date range filter
3. Sort by date descending (most recent first).
4. Calculate summary stats for filtered results:
   - Total transactions, total buys, total sells
   - Total value bought, total value sold
   - Total fees paid
5. Skip to Output.

## Phase 3E: Execute — Allocation

1. Read `.lifestack/data/finance/portfolio.md`.
2. Get current holdings (same aggregation as Phase 3B).
3. Group holdings by asset_class.
4. Calculate per-class:
   - Total cost basis in that class
   - Actual allocation percentage: (class_total / portfolio_total) * 100
5. If allocation targets exist in the data file (Allocation Targets table):
   - Load target percentages per asset class.
   - Calculate drift: actual_pct - target_pct for each class.
   - Flag significant drift (> 5 percentage points).
6. Sort by actual allocation descending.
7. Skip to Output.

## Phase 4: Create Checkpoint (writes only)

For log operations:

1. Before writing the modified portfolio.md, save a checkpoint:
   - Copy current portfolio.md to `.lifestack/data/finance/.checkpoints/portfolio-{timestamp}.md`
   - Create the .checkpoints directory if it doesn't exist.
2. Write the updated portfolio.md.

## Phase 5: Log

Append an entry to `.lifestack/audit/{YYYY-MM-DD}.md`:

```markdown
- [{HH:MM}] finance/portfolio: {command} — {summary}
```

Examples:
- `[09:00] finance/portfolio: log — Logged BUY 10 AAPL @ $185.50 (total: $1,860.00, fees: $5.00)`
- `[09:05] finance/portfolio: holdings — Displayed 6 holdings (total cost basis: $38,400)`
- `[09:10] finance/portfolio: performance — Portfolio P&L: +$12,500 (+32.5% overall)`
- `[09:15] finance/portfolio: history — Showed 24 transactions for AAPL (12 buys, 2 sells)`
- `[09:20] finance/portfolio: allocation — Allocation: stocks 55%, crypto 25%, bonds 15%, cash 5% (stocks under target by 5%)`

## Output

Format the result based on the command:

**log**: Confirm the logged transaction.
```
Transaction logged:
  Date:        2026-02-28
  Type:        BUY
  Asset:       AAPL (stocks)
  Quantity:    10 shares
  Price:       $185.50 / share
  Fees:        $5.00
  Total cost:  $1,860.00

  Updated holdings: 60 AAPL (avg cost basis: $158.33)
```

For sells:
```
Transaction logged:
  Date:        2026-02-28
  Type:        SELL
  Asset:       AAPL (stocks)
  Quantity:    10 shares
  Price:       $195.00 / share
  Fees:        $5.00
  Net proceeds: $1,945.00
  Realized gain: +$371.70 (+23.5% on cost basis of $158.33/share)

  Remaining holdings: 50 AAPL (avg cost basis: $158.33)
```

**holdings**: Show current holdings summary.
```
Portfolio Holdings

  Asset           Class     Qty      Avg Cost    Total Cost   Allocation
  ─────────────────────────────────────────────────────────────────────────
  Apple (AAPL)    stocks    50       $158.33     $7,916.50    20.6%
  Bitcoin (BTC)   crypto    0.5      $45,000     $22,500.00   58.6%
  S&P 500 (VOO)   etfs     20       $420.00     $8,400.00    21.9%
  ─────────────────────────────────────────────────────────────────────────
  Total                                          $38,816.50   100%

  3 positions across 3 asset classes
```

**performance**: Show P&L breakdown.
```
Portfolio Performance

  Asset           Qty    Cost Basis   Current Val   Unrealized     Return
  ─────────────────────────────────────────────────────────────────────────
  Apple (AAPL)    50     $7,916.50    $9,275.00     +$1,358.50    +17.2%
  Bitcoin (BTC)   0.5    $22,500.00   $48,500.00    +$26,000.00   +115.6%
  S&P 500 (VOO)  20     $8,400.00    $8,900.00     +$500.00      +5.9%
  ─────────────────────────────────────────────────────────────────────────
  Unrealized                          $66,675.00    +$27,858.50   +71.8%

  Realized gains (closed positions):  +$371.70
  Total return (realized + unrealized): +$28,230.20 (+72.7%)
```

**history**: Show filtered transaction history.
```
Transaction History — AAPL

  Date         Type   Qty    Price      Fees    Total
  ────────────────────────────────────────────────────────
  2026-02-28   SELL   10     $195.00    $5.00   $1,945.00
  2026-02-15   BUY    10     $185.50    $5.00   $1,860.00
  2026-01-10   BUY    20     $172.00    $5.00   $3,445.00
  2025-11-05   BUY    30     $150.00    $5.00   $4,505.00
  ────────────────────────────────────────────────────────
  4 transactions · 3 buys ($9,810.00) · 1 sell ($1,945.00)
  Total fees: $20.00
```

**allocation**: Show allocation with target comparison.
```
Portfolio Allocation

  Asset Class     Actual    Target    Drift
  ────────────────────────────────────────────────
  crypto          58.6%     10.0%     +48.6% ⚠ significantly over
  etfs            21.9%     —         —
  stocks          20.6%     60.0%     -39.4% ⚠ significantly under
  bonds            0.0%     20.0%     -20.0% ⚠ significantly under
  cash             0.0%     10.0%     -10.0% ⚠ under
  ────────────────────────────────────────────────

  Drift analysis:
    4 of 4 target classes have significant drift (> 5%).
    Crypto is heavily overweight relative to target.
    Consider rebalancing: reduce crypto exposure, add stocks and bonds.

  Note: Allocation is based on cost basis, not current market value.
  Run `portfolio performance` for market-value-based allocation.
```
