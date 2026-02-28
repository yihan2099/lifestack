---
name: nutrition-log
description: Log meals, track daily nutrition, and review eating patterns
argument-hint: "<log|daily|summary> [--meal breakfast|lunch|dinner|snack] [--description '...'] [--calories N] [--date YYYY-MM-DD] [--range 7d|30d]"
metadata:
  domain: health
  permission-tier: read-write
  openclaw: {"emoji": "🍽️", "requires": {"bins": []}}
---

<!-- SAFETY PREAMBLE — DO NOT SUMMARIZE, COMPACT, OR REMOVE -->
This skill operates at permission tier: read-write.
It may create and modify files only within .lifestack/data/health/.
It must NEVER delete user data without explicit confirmation.
Health data is PRIVATE and stored locally in .lifestack/data/health/. It is NEVER sent to external services.
All actions are logged to .lifestack/audit/YYYY-MM-DD.md.
<!-- END SAFETY PREAMBLE -->

## Input Parsing

Parse the user's command into one of three actions:

| Command | Required Args | Optional Args |
|---------|--------------|---------------|
| `log` | `--meal`, `--description` | `--calories`, `--notes`, `--date` |
| `daily` | (none) | `--date` |
| `summary` | (none) | `--range` |

Defaults:
- `--date` defaults to today (YYYY-MM-DD)
- `--calories` is optional — do not pressure the user to track calories
- `--range` defaults to `7d`

Valid meal types: `breakfast`, `lunch`, `dinner`, `snack`.

If the user provides natural language, extract values. Examples:
- "had oatmeal with berries for breakfast" -> `log --meal breakfast --description "oatmeal with berries"`
- "lunch: chicken salad, about 500 cal" -> `log --meal lunch --description "chicken salad" --calories 500`
- "snacked on almonds" -> `log --meal snack --description "almonds"`

## Phase 1: Data File Setup

Data file: `.lifestack/data/health/nutrition.md`

If the file does not exist, create it with this header:

```markdown
# Nutrition Log

| date | meal_type | description | calories | notes |
|------|-----------|-------------|----------|-------|
```

Read the existing file content for all commands.

## Phase 2: Execute Command

### log

1. Validate all inputs:
   - `meal` must be one of the valid meal types
   - `description` must be non-empty
   - `calories` if provided must be a positive integer
   - `date` must be valid YYYY-MM-DD and not in the future
2. Append a new row to the nutrition table:
   ```
   | YYYY-MM-DD | breakfast | oatmeal with berries | - | notes |
   ```
   Use `-` for calories if not provided (never estimate or fabricate calorie counts).
3. Write audit entry: `[HH:MM] nutrition-log: logged {meal_type} — {description}`

### daily

1. Filter entries for the specified date (default today)
2. Display all meals for that day in chronological order (breakfast -> lunch -> dinner -> snack)
3. Show:
   - Each meal with description and calories (if tracked)
   - Total calories for the day (if any meals have calorie data)
   - Meal count and which meals are logged
   - Missing meals: note which standard meals (breakfast, lunch, dinner) are not yet logged

### summary

1. Filter entries by `--range`
2. Calculate and display:
   - Total meals logged in period
   - Average meals per day
   - Meal type breakdown (count per type)
   - Days with all three main meals logged vs. days with gaps
   - If calorie data exists for enough entries (>50%):
     - Average daily calories
     - Calorie range (min-max day)
   - Eating patterns:
     - Most common meal descriptions (top 5 repeated items)
     - Snack frequency
   - No judgmental language — present data neutrally

## Output

Format all output as clean markdown.

For `log`, confirm the entry:
```
Logged: breakfast — oatmeal with berries (2026-02-28)
Today so far: 2 of 3 main meals logged
```

For `daily`, show a clean day view:
```
## 2026-02-28 — Meals

| Meal | Description | Calories |
|------|-------------|----------|
| Breakfast | oatmeal with berries | - |
| Lunch | chicken salad | 500 |
| Dinner | (not logged) | - |
| Snack | almonds | 160 |

Total tracked calories: 660
```

For `summary`, show a structured report with neutral, non-judgmental language.
