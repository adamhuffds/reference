# PostgreSQL Setup — Ubuntu

Docs: https://www.postgresql.org/docs/current/

## Install

```bash
sudo apt update
sudo apt install postgresql postgresql-client
```
- `postgresql` — the database server itself
- `postgresql-client` — the `psql` command-line client and related tools

## Manage the Service

```bash
sudo systemctl status postgresql     # check if it's running
sudo systemctl start postgresql        # start it if it isn't
sudo systemctl enable postgresql         # start automatically on boot
```

## Connecting to a Database

```bash
# Explicit connection parameters
psql -h localhost -U postgres -d my_database
```
- `-h` — host (`localhost` for a local install)
- `-U` — username to connect as
- `-d` — database name to connect to

```bash
# Shorthand: connects as your current Linux username, to a database
# with the same name as your Linux username (if one exists)
psql
# equivalent to:
psql -U your_username -d your_username

# Connect as your Linux user to a specific database
psql my_database
psql -U your_username -d my_database

# Connect as the default 'postgres' superuser
sudo -u postgres psql

# Standard dev-machine workflow: connect as postgres user to a specific db
sudo -u postgres psql my_database
```
`sudo -u postgres` runs the command as the `postgres` Linux system user,
which by default has a matching PostgreSQL role with full admin rights —
common for local dev setups before you've created your own role.

## Confirming You're Connected

Your prompt changes to show the connected database:
```
postgres=#
```
or
```
my_database=#
```

## Allowing Login Without `sudo` (Dev Convenience)

```sql
CREATE ROLE your_linux_username WITH LOGIN SUPERUSER;
```
Run this once (from inside `sudo -u postgres psql`) to create a PostgreSQL
role matching your Linux username, so plain `psql` works without `sudo -u
postgres` every time. Fine for local dev; avoid `SUPERUSER` on anything
resembling production — see `sql-database-introspection.md` for scoped role
creation.