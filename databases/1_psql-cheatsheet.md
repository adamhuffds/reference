# psql Meta-Commands Cheat Sheet

Docs: https://www.postgresql.org/docs/current/app-psql.html

Meta-commands (prefixed with `\`) are `psql`-specific — they're not SQL, and
only work inside an interactive `psql` session.

## Most Commonly Used

```
\pset pager off    # turn off the pager (no more "press q to quit" on long output) — cleaner for scripting/copying
\l                    # list all databases
\c dbname               # connect to a different database
\dt                        # list tables in the current schema
\d table_name                # describe a table's columns, types, indexes, constraints
\q                              # quit psql
\?                                # help — list all meta-commands
\h                                  # help for SQL commands (type a command after, e.g. \h SELECT)
\conninfo                             # show info about the current connection
```

## Listing Objects

```
\du               # list roles/users
\dn                 # list schemas
\dt *.*               # list tables across all schemas
\dt public.*             # list tables in the 'public' schema (swap in any schema name)
\dt schema_name.*           # list tables in a specific schema
\dt *pattern*                  # find tables matching a pattern
\d+ table_name                    # describe a table with extra detail (size, storage, etc.)
\di                                  # list indexes
\di table_name*                        # list indexes for a specific table
\dv                                       # list views
\ds                                          # list sequences
\df                                             # list functions
```

## Getting Help

```
\h SELECT          # SQL command syntax help — swap SELECT for any SQL command
```

## Display & Session Settings

```
\x on              # expanded display on — one column per line, much easier to read wide rows
\x off               # expanded display off, back to normal table layout
\timing on              # show how long each query took to run
\timing off                # turn off query timing
\echo :HOST :PORT :USER :DBNAME   # print current connection variables
```

## Running Scripts & Shell Commands

```
\i /path/to/file.sql          # execute a .sql file
\! bash_command                  # run a shell command without leaving psql
\set ON_ERROR_STOP on               # stop executing a script immediately if a statement errors
```
`\set ON_ERROR_STOP on` is worth setting at the top of any `.sql` script you
run with `\i` — without it, psql keeps executing subsequent statements even
after one fails, which can leave a script partially, silently applied.

## Import / Export CSV

```
\copy (SELECT * FROM table_name) TO '/tmp/out.csv' CSV HEADER    # export a query's results to CSV
\copy table_name FROM '/tmp/in.csv' CSV HEADER                     # import a CSV into a table
```
- `\copy` runs client-side (through psql), unlike the SQL `COPY` command
  which runs server-side and requires server filesystem permissions — `\copy`
  is generally the more convenient option for local dev work since it can
  read/write files anywhere your own user has access to, not just paths the
  Postgres server process can reach.
- `HEADER` tells it the first row is column names, not data

Docs: https://www.postgresql.org/docs/current/app-psql.html#APP-PSQL-META-COMMANDS-COPY