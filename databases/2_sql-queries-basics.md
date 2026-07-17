# SQL Query Basics (PostgreSQL)

Docs: https://www.postgresql.org/docs/current/sql-select.html

## Selecting Columns

```sql
SELECT column1, column2
FROM table_name;

SELECT *
FROM table_name;                -- all columns

SELECT *
FROM table_name
LIMIT 10;                        -- first 10 rows only
```

## Filtering with WHERE

```sql
SELECT *
FROM table_name
WHERE column = 'value';
```

Common comparison operators: `=`, `!=` (or `<>`), `>`, `>=`, `<`, `<=`

```sql
-- Multiple conditions
WHERE a = 1 AND b > 5
WHERE a = 1 OR b IS NULL

-- Pattern matching
WHERE name LIKE 'A%'       -- case-sensitive, % = wildcard for any characters
WHERE name ILIKE '%test%'    -- case-insensitive version of LIKE

-- IN / BETWEEN
WHERE status IN ('open', 'closed')
WHERE created_at BETWEEN '2025-01-01' AND '2025-12-31'
```

### NULL Handling
```sql
WHERE column IS NULL
WHERE column IS NOT NULL
```
`WHERE column = NULL` never matches anything — `NULL` isn't a value that can
be equal to another value in SQL's three-valued logic (`TRUE`/`FALSE`/
`UNKNOWN`), so always use `IS NULL`/`IS NOT NULL` instead of `= NULL`.

## Sorting & Pagination

```sql
ORDER BY created_at DESC
ORDER BY last_name ASC, first_name ASC     -- sort by multiple columns, in priority order

LIMIT 10
OFFSET 20

-- Pagination pattern
LIMIT 10 OFFSET 10 * (page - 1)
```

## DISTINCT

```sql
SELECT DISTINCT status
FROM orders;
```
Returns only unique values, removing duplicate rows from the result.

## Aliases

```sql
SELECT c.name AS customer_name
FROM customers c;
```
`AS` renames a column or table in the output/reference — `c` here is a table
alias, letting you write `c.name` instead of `customers.name` throughout the
query.

## Aggregates

```sql
SELECT COUNT(*) FROM table_name;
SELECT AVG(score) FROM table_name;
SELECT MIN(price), MAX(price) FROM table_name;
```

## GROUP BY

```sql
SELECT category, COUNT(*)
FROM products
GROUP BY category;

-- Filter on the grouped results (not the raw rows) with HAVING
SELECT category, COUNT(*)
FROM products
GROUP BY category
HAVING COUNT(*) > 10;
```
`WHERE` filters rows before grouping; `HAVING` filters groups after
aggregation — use `HAVING` specifically when your condition involves an
aggregate function like `COUNT()`, `AVG()`, etc.

## Query Execution Order (Mental Model)

SQL is *written* top-down, but *executes* in roughly this order:
```
FROM → JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```
This explains a few things that otherwise seem odd — e.g. why you can't
reference a `SELECT`-defined alias inside a `WHERE` clause (WHERE runs
before SELECT), but you generally can in `ORDER BY` (which runs after).

See `sql-queries-joins-advanced.md` for JOINs, subqueries, and
INSERT/UPDATE/DELETE.