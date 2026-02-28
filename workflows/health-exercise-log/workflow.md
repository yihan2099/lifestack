# Workflow: Health Exercise Log

## Run Command

```
You are testing the health/exercise skill. Perform the following operations:

1. Log the following workouts in sequence:
{{workout_entries}}

2. After all workouts are logged, check the current streak count
3. Request a summary of all logged workouts including:
   - Total workouts logged
   - Total duration (minutes)
   - Activity breakdown
   - Current streak (consecutive active days)

Use the health/exercise.md skill for all operations. Report each logged workout confirmation and the final summary.
```

## Success Criteria

- Output contains confirmation for each logged workout entry
- Output contains the correct streak count matching {{expected_streak}}
- Output contains total workout count matching {{expected_total}}
- Output contains total duration in minutes
- Output contains activity breakdown by type
- Summary statistics are consistent with the logged entries

## Fixtures

See `fixtures.md` for test data. Each fixture substitutes:
- `workout_entries` — list of workouts to log (date, activity, duration, intensity)
- `expected_streak` — expected consecutive active days
- `expected_total` — expected total number of workouts

## Constraints

- Timeout: 120000ms
- Max turns: 20
