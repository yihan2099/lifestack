---
name: mood-journal
description: Private mood tracking and journaling with pattern analysis
argument-hint: "<entry|mood|reflect|patterns> [--mood 1-10] [--energy 1-10] [--tags work,social,creative] [--range 7d|30d]"
metadata:
  domain: health
  permission-tier: read-write
  openclaw: {"emoji": "📝", "requires": {"bins": []}}
---

<!-- SAFETY PREAMBLE — DO NOT SUMMARIZE, COMPACT, OR REMOVE -->
This skill operates at permission tier: read-write.
It may create and modify files only within .lifestack/data/health/.
It must NEVER delete user data without explicit confirmation.
Health data is PRIVATE and stored locally in .lifestack/data/health/. It is NEVER sent to external services.
All actions are logged to .lifestack/audit/YYYY-MM-DD.md.

This skill handles deeply personal data. Journal entries and mood data are NEVER sent to external services, NEVER included in API calls to remote models, and NEVER shared with other skills unless explicitly requested by the user.

Audit log entries for this skill record only the action type (e.g., "journal entry added") and NEVER include the content of entries, mood values, or any personal text.
<!-- END SAFETY PREAMBLE -->

## Input Parsing

Parse the user's command into one of four actions:

| Command | Required Args | Optional Args |
|---------|--------------|---------------|
| `entry` | (free-form text) | `--mood`, `--energy`, `--tags`, `--date` |
| `mood` | `--mood` | `--note`, `--energy`, `--date` |
| `reflect` | (none) | `--date` |
| `patterns` | (none) | `--range` |

Defaults:
- `--date` defaults to today (YYYY-MM-DD)
- `--mood` scale: 1 (very low) to 10 (excellent)
- `--energy` scale: 1 (exhausted) to 10 (highly energized)
- `--range` defaults to `30d`

If the user provides natural language, extract values. Examples:
- "feeling great today, productive morning at work" -> `entry --mood 8 --energy 8 --tags work,productive` with the text as the journal body
- "mood 6" -> `mood --mood 6`
- "had a tough day, energy is low" -> `entry --mood 4 --energy 3` with the text as the journal body

## Phase 1: Data File Setup

Data file: `.lifestack/data/health/journal.md`

If the file does not exist, create it with this structure:

```markdown
# Mood & Journal

## Entries

```

The journal uses a different format from other health skills because entries can be multi-line. Each entry is a section:

```markdown
### YYYY-MM-DD HH:MM

- **Mood**: 7/10
- **Energy**: 6/10
- **Tags**: work, creative

Free-form journal text goes here. Can be multiple lines
and paragraphs.

---
```

For quick mood logs (no journal text), use a compact format appended to a daily mood table at the top of the file:

```markdown
## Quick Moods

| date | time | mood | energy | note |
|------|------|------|--------|------|
```

## Phase 2: Execute Command

### entry

1. Validate inputs:
   - `mood` if provided must be integer 1-10
   - `energy` if provided must be integer 1-10
   - `date` must be valid YYYY-MM-DD
2. Create a new journal entry section with the current timestamp
3. Include mood, energy, and tags if provided
4. Include the full free-form text as the entry body
5. Append the entry to the Entries section of the journal file
6. Write audit entry (content-free): `[HH:MM] mood-journal: journal entry added`

### mood

1. Validate:
   - `mood` must be integer 1-10
   - `energy` if provided must be integer 1-10
2. Append a row to the Quick Moods table:
   ```
   | YYYY-MM-DD | HH:MM | 7 | 6 | optional one-line note |
   ```
3. Write audit entry (content-free): `[HH:MM] mood-journal: mood logged`

### reflect

1. Read recent journal entries (last 7 days)
2. Present 3 guided reflection prompts. Rotate through these sets:

   **Set A:**
   - What went well today/this week?
   - What's weighing on your mind?
   - What are you grateful for?

   **Set B:**
   - What gave you energy recently?
   - What drained your energy?
   - What would you do differently?

   **Set C:**
   - What are you looking forward to?
   - What challenge are you working through?
   - What small win can you celebrate?

3. If recent entries exist, reference them gently: "You mentioned [tag] a few times recently — want to explore that?"
4. Wait for the user's response and log it as a new entry with tag `reflection`

### patterns

1. Read all entries and quick moods within `--range`
2. Calculate and display:
   - Average mood and energy over the period
   - Mood trend: improving, declining, or stable (compare first half to second half)
   - Energy trend: improving, declining, or stable
   - Most common tags and their average mood/energy
   - Day-of-week patterns (e.g., "Mood tends to be higher on weekends")
   - If exercise data exists in `.lifestack/data/health/exercise.md`:
     - Compare mood on exercise days vs. rest days
   - If sleep data exists in `.lifestack/data/health/sleep.md`:
     - Correlate sleep quality/duration with next-day mood
3. Present insights neutrally and compassionately — never judgmental

## Output

Format all output as clean markdown.

For `entry`, confirm briefly:
```
Journal entry saved (2026-02-28 14:30)
Mood: 7/10 | Energy: 6/10 | Tags: work, creative
```

For `mood`, confirm briefly:
```
Mood logged: 7/10 (2026-02-28 14:30)
```

For `reflect`, present the prompts warmly and wait for input.

For `patterns`, show a structured report. Use neutral, compassionate language. Never moralize about mood scores. Present correlations as observations, not prescriptions:
- Say: "Mood was higher on days with exercise logged (avg 7.2 vs 5.8)"
- Not: "You should exercise more to improve your mood"
