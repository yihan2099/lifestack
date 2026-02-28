# Fixer: Health Exercise Log

Common failure patterns and remediation steps.

## "Streak count wrong"

**Cause**: The streak calculation does not correctly handle rest days (gaps) between active days. A streak should count consecutive calendar days with at least one logged workout.
**Fix**: Verify the streak calculation logic in the exercise skill. The algorithm should:
1. Sort all workout dates chronologically
2. Count backwards from the most recent workout date
3. A streak breaks when there is a calendar day with no logged workout between two active days
4. Rest days explicitly logged should not count as streak-breakers unless they represent a gap in the date sequence

## "Duration format error"

**Cause**: The workout duration is not being parsed as an integer representing minutes. The skill may be receiving strings like "45 min" or "1:00" instead of a plain integer.
**Fix**: Ensure duration is always parsed and stored as an integer (minutes). Strip unit suffixes ("min", "minutes") during parsing. If duration is provided in hours:minutes format, convert to total minutes before storing.

## "Data not persisted"

**Cause**: The exercise skill is not writing workout data to the correct file path, or the data directory does not exist.
**Fix**: Ensure the exercise skill writes data to `.lifestack/data/health/exercise.md` (or the configured data path). Create the directory if it does not exist:

```bash
mkdir -p .lifestack/data/health/
```

Verify that each log operation appends to or updates the data file rather than overwriting it.

## "Duplicate workout entry"

**Cause**: The same workout was logged twice for the same date and activity, inflating totals.
**Fix**: The exercise skill should check for existing entries matching the same date and activity type before creating a new entry. If a duplicate is detected, update the existing entry rather than creating a new one.

## "Summary totals do not match"

**Cause**: The summary calculation is counting rest days or failed log entries in the totals.
**Fix**: Ensure the summary only aggregates successfully confirmed workout entries. Rest days should be excluded from total workout count and total duration. Cross-check the count of confirmation messages against the summary total.
