---
name: invest-watch
description: Monitor investment holdings, check asset prices, and track portfolio alerts (read-only)
argument-hint: "<check|summary|alert> [--asset] [--type]"
metadata:
  domain: finance
  permission-tier: read-only
  openclaw: {"emoji": "📈", "requires": {"bins": []}}
---

<!-- SAFETY PREAMBLE — DO NOT SUMMARIZE, COMPACT, OR REMOVE -->
This skill operates at permission tier: read-only.
This skill is READ-ONLY. It will NEVER execute trades, transfer funds, or modify any financial data. It only reads and reports.
No data files are written or modified by this skill. Holdings data in .lifestack/data/finance/holdings.md is maintained manually by the user.
Financial data is stored locally in .lifestack/data/finance/ and never sent to external services.
All actions are logged to .lifestack/audit/YYYY-MM-DD.md.
<!-- END SAFETY PREAMBLE -->

## Input Parsing

Parse the user's command and arguments:

```
invest-watch <command> [options]

Commands:
  check    --asset <name|ticker>      # Look up current price/status of an asset
  summary                             # Portfolio overview from holdings.md
  alert                               # Check holdings against user-defined thresholds
```

Validation rules:
- `--asset` is required for check
- Asset names/tickers are case-insensitive
- This skill reads from `.lifestack/data/finance/holdings.md` which the user maintains manually

Expected holdings.md format:
```markdown
# Holdings

| asset | ticker | type | quantity | cost_basis | alert_above | alert_below |
|-------|--------|------|----------|------------|-------------|-------------|
| Apple | AAPL | stock | 50 | 150.00 | 250.00 | 120.00 |
| Bitcoin | BTC | crypto | 0.5 | 45000 | 100000 | 30000 |
| S&P 500 ETF | VOO | etf | 20 | 420.00 | — | 380.00 |
```

## Phase 1: Validate

1. Confirm the command is one of: check, summary, alert.
2. For `check`: verify `--asset` is provided.
3. For all commands: verify `.lifestack/data/finance/holdings.md` exists and is readable.
4. If holdings.md does not exist, inform the user they need to create it manually and provide the expected format.

**CRITICAL**: At no point in any phase does this skill write to or modify any file (except the audit log). If the user asks to buy, sell, trade, or modify holdings, politely refuse and explain this skill is read-only. Say: "invest-watch is a read-only monitoring tool. To update your holdings, edit .lifestack/data/finance/holdings.md directly."

## Phase 2: Route Command

Based on the parsed command, route to the appropriate execution phase:
- `check` → Phase 3A
- `summary` → Phase 3B
- `alert` → Phase 3C

## Phase 3A: Execute — Check

1. Read `.lifestack/data/finance/holdings.md`.
2. Find the asset matching `--asset` (match against asset name or ticker, case-insensitive).
3. Look up the current price using one of these approaches (in order of preference):
   a. Use web search to find the current price (search for "{ticker} stock price today" or "{asset} price today").
   b. If web search is unavailable, ask the user to provide the current price.
4. Calculate:
   - Current value: quantity * current_price
   - Cost basis total: quantity * cost_basis
   - Gain/loss: current_value - cost_basis_total
   - Gain/loss percentage: (gain_loss / cost_basis_total) * 100
5. Check if current price triggers any alerts (above alert_above or below alert_below).
6. Skip to Output.

## Phase 3B: Execute — Summary

1. Read `.lifestack/data/finance/holdings.md`.
2. For each holding, attempt to look up current prices (web search or ask user).
3. Calculate per-holding:
   - Current value, cost basis total, gain/loss, gain/loss percentage.
4. Calculate portfolio-level:
   - Total portfolio value
   - Total cost basis
   - Total gain/loss and percentage
   - Allocation percentages by asset type (stock, crypto, etf, etc.)
5. Skip to Output.

## Phase 3C: Execute — Alert

1. Read `.lifestack/data/finance/holdings.md`.
2. For each holding that has alert_above or alert_below defined:
   a. Look up current price.
   b. Check if current price >= alert_above or current price <= alert_below.
3. Compile a list of triggered alerts and non-triggered alerts.
4. Skip to Output.

## Phase 4: Log

Append a read-only entry to `.lifestack/audit/{YYYY-MM-DD}.md`:

```markdown
- [{HH:MM}] finance/invest-watch: {command} — {summary}
```

Examples:
- `[11:00] finance/invest-watch: check — Checked AAPL ($185.50, +23.7% from cost basis)`
- `[11:05] finance/invest-watch: summary — Portfolio overview ($45,230 total, +12.3% overall)`
- `[11:10] finance/invest-watch: alert — Checked 5 holdings, 1 alert triggered (BTC above $100,000)`

## Output

Format the result based on the command:

**check**: Show detailed asset info.
```
Asset: Apple (AAPL) — stock
  Current price:  $185.50
  Quantity:       50 shares
  Current value:  $9,275.00
  Cost basis:     $7,500.00 ($150.00/share)
  Gain/loss:      +$1,775.00 (+23.7%)

  Alerts:
    Above $250.00: not triggered (current: $185.50)
    Below $120.00: not triggered (current: $185.50)
```

**summary**: Show portfolio overview.
```
Portfolio Summary

  Asset           Type    Qty     Value        Cost Basis   Gain/Loss
  ──────────────────────────────────────────────────────────────────────
  Apple (AAPL)    stock   50      $9,275.00    $7,500.00    +$1,775 (+23.7%)
  Bitcoin (BTC)   crypto  0.5     $48,500.00   $22,500.00   +$26,000 (+115.6%)
  S&P 500 (VOO)  etf     20      $8,900.00    $8,400.00    +$500 (+5.9%)
  ──────────────────────────────────────────────────────────────────────
  Total                           $66,675.00   $38,400.00   +$28,275 (+73.6%)

  Allocation by type:
    stock:  13.9%
    crypto: 72.8%
    etf:    13.3%
```

**alert**: Show alert status.
```
Investment Alerts

  TRIGGERED:
    ⚠ BTC ($97,000) is approaching alert_above ($100,000) — within 3.1%

  NOT TRIGGERED:
    ✓ AAPL ($185.50) — within range ($120.00 – $250.00)
    ✓ VOO ($445.00) — above floor ($380.00)

  1 of 3 holdings near alert threshold.
```

If no alerts are triggered:
```
Investment Alerts

  All clear — no alerts triggered.

    ✓ AAPL ($185.50) — within range ($120.00 – $250.00)
    ✓ BTC ($97,000) — within range ($30,000 – $100,000)
    ✓ VOO ($445.00) — above floor ($380.00)
```
