# Fixer: Finance Budget CRUD

Common failure patterns and remediation steps.

## "Budget not found"

**Cause**: Attempting to check, update, or delete a budget before it has been created.
**Fix**: Ensure the create step completed successfully before proceeding to check/update/delete. Re-run the create operation if the previous attempt failed silently.

## "Permission denied"

**Cause**: The budget skill requires a specific permission tier that has not been granted.
**Fix**: Verify the permission tier in the skill metadata (`finance/budget.md` frontmatter). Ensure the agent session has the required tier before invoking budget operations.

## "Data file missing"

**Cause**: The data storage directory does not exist yet.
**Fix**: Create the `.lifestack/data/finance/` directory before running budget operations. The budget skill expects this path to exist for persisting budget data.

```bash
mkdir -p .lifestack/data/finance/
```

## "Checkpoint failed"

**Cause**: The budget skill depends on the checkpoint system for state tracking, but the checkpoint skill is not available.
**Fix**: Ensure `core/checkpoint.md` skill is present and accessible in the skill registry. The budget CRUD flow uses checkpoints to track operation state between turns.
