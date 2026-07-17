# SQL — Database Introspection, Transactions & Admin (PostgreSQL)

Docs: https://www.postgresql.org/docs/current/information-schema.html

These are SQL-based alternatives to the `psql` meta-commands in
`psql-cheatsheet.md` (e.g. `\dt`, `\d`) — useful when you need this
information from within application code or a script, not just interactively.

## Inspecting Database Structure

```sql
-- List tables
SELECT table_schema, table_name
FROM information_schema.tables
WHERE table_type = 'BASE TABLE'
AND table_schema NOT IN ('pg_catalog', 'information_schema');

-- Show columns in a table
SELECT column_name, data_type, is_nullable, column_default
FROM information_schema.columns
WHERE table_name = 'table_name';

-- Show constraints on a table
SELECT conname, contype
FROM pg_constraint
WHERE conrelid = 'table_name'::regclass;

-- Find columns matching a name pattern, across all tables
SELECT table_schema, table_name, column_name, data_type
FROM information_schema.columns
WHERE column_name ILIKE '%pattern%';
```

## "What Am I Looking At" Commands

```sql
SELECT current_database();     -- which database is this connection using
SELECT current_user;             -- which role/user am I connected as
SHOW search_path;                  -- schema resolution order for unqualified table names
```

## Transactions

```sql
BEGIN;
-- do stuff (INSERT/UPDATE/DELETE statements)
COMMIT;      -- make the changes permanent
-- or
ROLLBACK;      -- undo everything since BEGIN
```
Wrapping multiple related statements in a transaction ensures they either
all succeed or all fail together — critical for multi-step operations (e.g.
transferring a value between two rows) where a partial failure would leave
data in an inconsistent state.
Docs: https://www.postgresql.org/docs/current/tutorial-transactions.html

## Destructive Operations

```sql
DROP DATABASE database_name;              -- permanently delete an entire database

TRUNCATE table_name RESTART IDENTITY;      -- clear all rows AND reset auto-increment counters
```
- `DROP DATABASE` cannot be undone outside of a backup restore — double
  check the database name before running
- `TRUNCATE ... RESTART IDENTITY` is faster than `DELETE FROM table_name`
  for clearing an entire table, and additionally resets any `SERIAL`/
  identity columns back to their starting value (plain `DELETE` does not)

## Roles & Permissions

```sql
-- Create a role with login access and a password
CREATE ROLE myuser WITH LOGIN PASSWORD 'pw';

-- Grant additional privileges as needed
ALTER ROLE myuser CREATEDB;         -- allow creating new databases
ALTER ROLE myuser SUPERUSER;         -- full admin rights (dev/local only — avoid in production)
```
Docs: https://www.postgresql.org/docs/current/sql-createrole.html

Prefer scoped grants (`CREATEDB`, specific table `GRANT`s) over blanket
`SUPERUSER` for any role beyond your own local dev convenience account —
see `postgresql-setup.md` for the dev-convenience `SUPERUSER` shortcut and
its trade-offs.