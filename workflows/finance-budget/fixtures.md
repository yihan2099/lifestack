# Fixtures: Finance Budget CRUD

## Fixture 1: Monthly Grocery Budget

- **ID**: `budget-monthly-groceries`
- **Input values**:
  - `budget_name`: "Monthly Groceries"
  - `amount`: 500
  - `updated_amount`: 550
  - `period`: monthly
  - `categories`: groceries, household
- **Expected success pattern**: Budget "Monthly Groceries" created with $500/monthly allocation across groceries and household categories. Update reflects $550. Deletion confirmed.

## Fixture 2: Quarterly Entertainment Budget

- **ID**: `budget-quarterly-entertainment`
- **Input values**:
  - `budget_name`: "Entertainment Fund"
  - `amount`: 300
  - `updated_amount`: 400
  - `period`: quarterly
  - `categories`: dining, movies, events
- **Expected success pattern**: Budget "Entertainment Fund" created with $300/quarterly allocation across dining, movies, and events categories. Update reflects $400. Deletion confirmed.

## Fixture 3: Yearly Vacation Budget

- **ID**: `budget-yearly-vacation`
- **Input values**:
  - `budget_name`: "Vacation Savings"
  - `amount`: 5000
  - `updated_amount`: 6000
  - `period`: yearly
  - `categories`: flights, hotels, activities
- **Expected success pattern**: Budget "Vacation Savings" created with $5000/yearly allocation across flights, hotels, and activities categories. Update reflects $6000. Deletion confirmed.
