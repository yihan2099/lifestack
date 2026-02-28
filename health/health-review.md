---
name: health-review
description: Generate a comprehensive weekly health review across all health data
argument-hint: "weekly [--range 7d|14d|30d] [--date YYYY-MM-DD]"
metadata:
  domain: health
  permission-tier: read-only
  openclaw: {"emoji": "📊", "requires": {"bins": []}}
---

<!-- SAFETY PREAMBLE — DO NOT SUMMARIZE, COMPACT, OR REMOVE -->
This skill operates at permission tier: read-only.
It may READ files within .lifestack/data/health/ but must NEVER create or modify them.
Health data is PRIVATE and stored locally in .lifestack/data/health/. It is NEVER sent to external services.
All actions are logged to .lifestack/audit/YYYY-MM-DD.md.
<!-- END SAFETY PREAMBLE -->

## Input Parsing

Parse the user's command:

| Command | Required Args | Optional Args |
|---------|--------------|---------------|
| `weekly` | (none) | `--range`, `--date` |

Defaults:
- `--range` defaults to `7d`
- `--date` defaults to today — the review covers the period ending on this date

This skill is read-only. It reads from all health data files but never modifies them.

## Phase 1: Data Collection

Read the following data files (skip any that don't exist — report which data sources are available):

| File | Domain |
|------|--------|
| `.lifestack/data/health/exercise.md` | Exercise |
| `.lifestack/data/health/sleep.md` | Sleep |
| `.lifestack/data/health/nutrition.md` | Nutrition |
| `.lifestack/data/health/journal.md` | Mood & Journal |

For each file that exists, parse the entries within the specified date range.

## Phase 2: Generate Review

Build a comprehensive review report with the following sections. Only include sections for which data exists.

### Exercise Summary
- Total workouts and total duration
- Breakdown by type
- Current streak (consecutive days with workouts)
- Comparison to previous period: "N more/fewer workouts than last week"
- Highlight: best workout (longest or highest intensity)

### Sleep Overview
- Average duration and quality
- Bedtime consistency
- Nights below 7 hours
- Best and worst night
- Comparison to previous period

### Nutrition Overview
- Meals logged per day (average)
- Meal coverage: how many days had all 3 main meals logged
- If calorie data exists: average daily calories
- Most frequent meals/foods

### Mood & Energy
- Average mood and energy scores
- Trend direction (improving/declining/stable)
- Most common tags/themes
- Note: reference mood data by trends only, never quote journal text in the review

### Cross-Domain Insights

This is the most valuable section. Look for correlations:

1. **Exercise + Sleep**: Did sleep quality improve on days following exercise?
2. **Exercise + Mood**: Was mood higher on exercise days vs. rest days?
3. **Sleep + Mood**: Did mood correlate with sleep duration or quality?
4. **Nutrition + Energy**: Any patterns between meal logging consistency and energy levels?
5. **Streaks**: What's the longest active streak across all domains?

Present these as observations, not prescriptions:
- Say: "Sleep quality averaged 4.2/5 on days after exercise vs. 3.1/5 on rest days"
- Not: "You should exercise more to sleep better"

### Recommendations

Based on the data, provide 2-3 actionable, specific suggestions:
- Only suggest things supported by the user's own data patterns
- Frame as experiments: "You might try..." or "Consider testing..."
- Keep recommendations brief and concrete

### Week at a Glance

End with a compact visual summary table:

```markdown
| Day | Exercise | Sleep | Meals | Mood |
|-----|----------|-------|-------|------|
| Mon | 30m run  | 7.5h  | 3/3   | 7    |
| Tue | -        | 6.0h  | 2/3   | 5    |
| Wed | 45m yoga | 8.0h  | 3/3   | 8    |
| ... | ...      | ...   | ...   | ...  |
```

Use `-` for days with no data in that domain.

## Phase 3: Audit

Write audit entry: `[HH:MM] health-review: weekly review generated ({range} ending {date})`

## Output

Format the complete review as a single markdown document with clear headers.

Open with a one-line summary: "Health Review: {start_date} to {end_date} — {N} data points across {M} domains"

Close with: "Next review: {date + range interval}"

Keep the tone neutral, encouraging, and data-driven. Celebrate consistency and streaks. Never moralize about gaps in data — people have busy weeks.
