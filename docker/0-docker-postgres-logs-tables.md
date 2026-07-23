# Docker PostgreSQL: Logs & Table Inspection

Quick reference for checking logs and inspecting tables in a PostgreSQL container running under Docker or Docker Compose.

---

## 1. Finding Your Container

```bash
# List all containers (running or stopped) built from the postgres image
docker ps -a --filter "ancestor=postgres" --format "{{.ID}}\t{{.Names}}\t{{.Status}}"
```

- `ps -a` — list all containers, not just running ones
- `--filter "ancestor=postgres"` — only show containers created from the `postgres` image
- `--format` — customize output columns (ID, name, status)

If you're using Docker Compose, the service name in your `docker-compose.yml` (e.g. `postgres`) can usually be used directly in place of a container name/ID.

---

## 2. Checking Logs

### Basic log output
```bash
docker logs postgres
```
- `logs` — fetch stdout/stderr captured from the container's main process

### With timestamps
```bash
docker logs -t postgres
```
- `-t` / `--timestamps` — prepend each line with an ISO 8601 timestamp

### Follow in real time
```bash
docker logs -f postgres
```
- `-f` / `--follow` — stream new log lines continuously (like `tail -f`)

### Last N lines only
```bash
docker logs --tail 50 postgres
```
- `--tail <n>` — limit output to the most recent N lines

### Since a relative/absolute time
```bash
docker logs --since 30m postgres
```
- `--since` — only show logs newer than a duration (`30m`, `2h`) or timestamp (`2024-01-01`)

### Docker Compose equivalent
```bash
docker-compose logs -f postgres
```
- `docker-compose logs` — same idea, scoped to a service defined in `docker-compose.yml`

---

## 3. Finding Connection Details

If you don't remember the username/database name set for the container:

```bash
docker exec postgres env | grep POSTGRES
```
- `docker exec` — run a command inside a running container
- `env | grep POSTGRES` — filter environment variables for `POSTGRES_USER`, `POSTGRES_DB`, `POSTGRES_PASSWORD`

Common defaults: username `postgres`, database `postgres` (unless overridden in `docker-compose.yml` or at container creation).

---

## 4. Listing Tables

### One-off command via `psql`
```bash
docker exec -it postgres psql -U your_username -d your_database -c "\dt"
```
- `-it` — interactive (`-i`) + pseudo-TTY (`-t`), needed for an interactive psql session
- `-U your_username` — connect as this Postgres role
- `-d your_database` — target database
- `-c "\dt"` — run `\dt` (list tables in current schema) then exit

### Interactive session
```bash
docker exec -it postgres psql -U your_username -d your_database
```
Once inside:
```sql
\dt          -- tables in current schema
\dt+         -- tables + size/description
\dt *.*      -- tables across all schemas
\dt public.* -- tables in a specific schema
\d table_name -- describe a table's columns/structure
\q           -- quit
```

### Via `information_schema` (portable SQL, not psql-specific)
```bash
docker exec -it postgres psql -U your_username -d your_database -c "
SELECT
    table_schema,
    table_name,
    table_type
FROM information_schema.tables
WHERE table_schema NOT IN ('pg_catalog', 'information_schema')
ORDER BY table_schema, table_name;
"
```
- `information_schema.tables` — ANSI-standard metadata view (works across most SQL databases, not just Postgres)
- excludes `pg_catalog` / `information_schema` — filters out internal system tables

### With size and row count
```bash
docker exec -it postgres psql -U your_username -d your_database -c "
SELECT
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size,
    pg_stat_get_live_tuples(c.oid) AS row_count
FROM pg_tables
LEFT JOIN pg_class c ON pg_tables.tablename = c.relname
WHERE schemaname NOT IN ('pg_catalog', 'information_schema')
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
"
```
- `pg_tables` — Postgres system catalog view of all tables
- `pg_size_pretty()` — converts bytes to human-readable units (KB/MB/GB)
- `pg_total_relation_size()` — total on-disk size including indexes + TOAST data
- `pg_stat_get_live_tuples()` — estimated live row count for a table

### Docker Compose equivalent
```bash
docker-compose exec postgres psql -U your_username -d your_database -c "\dt"
```

---

## Reference Docs
- [Docker CLI: `docker logs`](https://docs.docker.com/reference/cli/docker/container/logs/)
- [Docker CLI: `docker exec`](https://docs.docker.com/reference/cli/docker/container/exec/)
- [Docker Compose CLI: `logs` / `exec`](https://docs.docker.com/reference/cli/docker/compose/)
- [PostgreSQL: `psql` reference](https://www.postgresql.org/docs/current/app-psql.html)
- [PostgreSQL: `information_schema.tables`](https://www.postgresql.org/docs/current/infoschema-tables.html)
- [PostgreSQL: `pg_tables` system view](https://www.postgresql.org/docs/current/view-pg-tables.html)
- [PostgreSQL: Database Object Size Functions](https://www.postgresql.org/docs/current/functions-admin.html#FUNCTIONS-ADMIN-DBSIZE)