# Workflow: Finance Budget CRUD

## Run Command

```
You are testing the finance/budget skill. Perform the following operations in sequence:

1. Create a budget with name "{{budget_name}}", amount ${{amount}}/{{period}}, categories: {{categories}}
2. Check the budget exists and verify its details
3. Update the budget amount to ${{updated_amount}}
4. List all budgets and confirm the updated budget appears
5. Delete the budget
6. List all budgets and confirm it was removed

Use the finance/budget.md skill for all operations. Report each step's result.
```

## Success Criteria

- Output contains "Budget created" or confirmation of budget creation
- Output contains the budget name "{{budget_name}}" after check
- Output contains "Budget updated" or confirmation of amount change to ${{updated_amount}}
- Output contains the budget in a list view
- Output contains "Budget deleted" or confirmation of removal
- Final list does not contain the deleted budget

## Fixtures

See `fixtures.md` for test data. Each fixture substitutes:
- `budget_name` — name of the budget
- `amount` — initial dollar amount
- `updated_amount` — new dollar amount after update
- `period` — monthly, quarterly, or yearly
- `categories` — comma-separated category list

## Constraints

- Timeout: 120000ms
- Max turns: 20
