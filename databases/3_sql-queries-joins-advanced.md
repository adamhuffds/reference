# SQL Queries — JOINs, Subqueries & Data Modification (PostgreSQL)

Docs: https://www.postgresql.org/docs/current/queries-table-expressions.html#QUERIES-JOIN

Prerequisite: `sql-queries-basics.md` for SELECT/WHERE/GROUP BY fundamentals.

## JOINs

### INNER JOIN — only rows with a match on both sides
```sql
SELECT o.id, c.name
FROM orders o
JOIN customers c ON o.customer_id = c.id;
```
`JOIN` alone defaults to `INNER JOIN` in PostgreSQL — they're equivalent,
but writing `INNER JOIN` explicitly can be clearer when reading a query
alongside `LEFT JOIN`s.

### LEFT JOIN — keep all rows from the left table, even without a match
```sql
SELECT c.name, o.id
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.id;
```
Unmatched rows from `customers` still appear, with `o.id` as `NULL`.

### Finding "missing" relations (a common LEFT JOIN use case)
```sql
SELECT c.name
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.id
WHERE o.id IS NULL;      -- customers with no orders at all
```

## Subqueries

```sql
SELECT *
FROM orders
WHERE customer_id IN (
    SELECT id
    FROM customers
    WHERE active = true
);
```
A query nested inside another — here, the inner query produces a list of
active customer IDs that the outer query filters against. Often rewritable
as a JOIN for performance on large tables, but subqueries are frequently
clearer to read/write for one-off filtering logic.

## CASE (Conditional Logic)

```sql
SELECT
    score,
    CASE
        WHEN score >= 90 THEN 'A'
        WHEN score >= 80 THEN 'B'
        ELSE 'C'
    END AS grade
FROM exams;
```
Evaluates top-to-bottom, stops at the first matching `WHEN` — order
conditions from most to least specific.

## Dates & Timestamps

```sql
NOW()             -- current timestamp
CURRENT_DATE        -- current date only, no time

-- Interval arithmetic
WHERE created_at > NOW() - INTERVAL '7 days'
```
Docs: https://www.postgresql.org/docs/current/functions-datetime.html

## Inserting Data

```sql
INSERT INTO table_name (col1, col2)
VALUES ('a', 123);

-- Multiple rows in one statement
INSERT INTO table_name (col1, col2)
VALUES
    ('a', 1),
    ('b', 2);
```

## Updating Data

```sql
UPDATE table_name
SET status = 'closed'
WHERE id = 5;
```
**Always run the equivalent `SELECT * FROM table_name WHERE id = 5;` first**
to confirm exactly which rows will be affected before running an `UPDATE` —
an `UPDATE` without a `WHERE` clause modifies every row in the table.

## Deleting Data

```sql
DELETE FROM table_name
WHERE id = 5;
```
Same caution applies — a `DELETE` with no `WHERE` clause removes every row.

## RETURNING (PostgreSQL-Specific)

```sql
INSERT INTO users (email)
VALUES ('a@test.com')
RETURNING id;
```
Returns the specified column(s) from the affected row(s) immediately, saving
a separate `SELECT` — also works with `UPDATE` and `DELETE`. Commonly used
in application code (e.g. the Flask/SQLAlchemy examples in
`flask-database-sqlalchemy.md`) to get a newly-inserted row's ID back in one
round trip.

## Common Mistakes to Avoid

- ❌ `WHERE x = NULL` — use `IS NULL` instead (see `sql-queries-basics.md`)
- ❌ `GROUP BY` missing a non-aggregated column that's also in `SELECT` —
  PostgreSQL will raise an error rather than silently guessing
- ❌ Forgetting `WHERE` in an `UPDATE`/`DELETE` — always test with `SELECT`
  first
- ❌ Joining without an `ON` condition — produces a cartesian product (every
  row from one table paired with every row from the other), rarely what's
  intended