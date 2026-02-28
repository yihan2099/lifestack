# Fixtures: Health Exercise Log

## Fixture 1: Daily Runner

- **ID**: `exercise-daily-runner`
- **Input values**:
  - `workout_entries`:
    - Day 1 (2026-02-20): Running, 30 min, moderate intensity
    - Day 2 (2026-02-21): Running, 35 min, moderate intensity
    - Day 3 (2026-02-22): Running, 40 min, high intensity
    - Day 4 (2026-02-23): Running, 30 min, low intensity (recovery)
    - Day 5 (2026-02-24): Running, 45 min, high intensity
    - Day 6 (2026-02-25): Running, 60 min, moderate intensity (long run)
    - Day 7 (2026-02-26): Running, 30 min, low intensity (easy)
  - `expected_streak`: 7
  - `expected_total`: 7
- **Expected success pattern**: 7 workout confirmations logged. Streak count is 7 consecutive days. Total duration is 270 minutes. Activity breakdown shows 100% running.

## Fixture 2: Gym Schedule (3x/week + yoga)

- **ID**: `exercise-gym-schedule`
- **Input values**:
  - `workout_entries`:
    - Mon (2026-02-16): Strength training, 60 min, high intensity
    - Tue (2026-02-17): Rest day (no workout)
    - Wed (2026-02-18): Strength training, 55 min, high intensity
    - Thu (2026-02-19): Rest day (no workout)
    - Fri (2026-02-20): Strength training, 60 min, moderate intensity
    - Sat (2026-02-21): Yoga, 45 min, low intensity
    - Sun (2026-02-22): Rest day (no workout)
  - `expected_streak`: 2 (Fri-Sat consecutive active days)
  - `expected_total`: 4
- **Expected success pattern**: 4 workout confirmations logged (rest days skipped). Streak is 2 (Friday + Saturday). Total duration is 220 minutes. Activity breakdown shows 3 strength training, 1 yoga.

## Fixture 3: Mixed Activities

- **ID**: `exercise-mixed-activities`
- **Input values**:
  - `workout_entries`:
    - Day 1 (2026-02-17): Running, 40 min, moderate intensity
    - Day 2 (2026-02-18): Swimming, 45 min, high intensity
    - Day 3 (2026-02-19): Rest day (no workout)
    - Day 4 (2026-02-20): Cycling, 60 min, moderate intensity
    - Day 5 (2026-02-21): Strength training, 50 min, high intensity
    - Day 6 (2026-02-22): Running, 35 min, low intensity
    - Day 7 (2026-02-23): Rest day (no workout)
    - Day 8 (2026-02-24): Swimming, 40 min, moderate intensity
    - Day 9 (2026-02-25): Cycling, 55 min, high intensity
    - Day 10 (2026-02-26): Strength training, 45 min, moderate intensity
  - `expected_streak`: 3 (Days 8-10 consecutive)
  - `expected_total`: 8
- **Expected success pattern**: 8 workout confirmations logged (2 rest days skipped). Streak is 3 (Feb 24-26). Total duration is 370 minutes. Activity breakdown shows 2 running, 2 swimming, 2 cycling, 2 strength training.
