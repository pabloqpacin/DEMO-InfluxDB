```sql
-- https://docs.influxdata.com/flux/v0/data-types/basic/null/
-- Check if a column value is null: In functions that iterate over rows (such as filter() or map()), use the exists logical operator to check if a column value is null.
-- Filter out rows with null values
-- (An empty string ("") is not a null value.)
|> filter(fn: (r) => exists r._value)
```

