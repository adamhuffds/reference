# Docker Compose Guide

Docs: https://docs.docker.com/compose/

Docker Compose runs **multiple containers together** as a single stack.
Instead of long `docker run` commands, you define services declaratively in
a YAML file (`docker-compose.yml`).

## Basic Structure

```yaml
services:
  service_name:
    image: image_name
```

Each service defined under `services:` gets:
- its own container
- a shared network with the other services in the same file
- a DNS hostname equal to the service name (see Networking below)

## Core Commands

```bash
docker compose up -d          # start the stack (-d = detached, runs in background)
docker compose up --build     # start the stack, rebuilding images first
docker compose down           # stop and remove containers, networks
docker compose down -v        # also remove named volumes (wipes persistent data)
docker compose ps             # list containers in this stack and their status
docker compose exec service_name bash   # open a shell inside a running service's container
```

## Volumes & Persistence

Containers are disposable; volumes persist data outside the container's
lifecycle.

```yaml
volumes:
  pgdata:
```
```yaml
services:
  postgres:
    image: postgres:16
    volumes:
      - pgdata:/var/lib/postgresql/data
```
This is a **named volume** — Docker manages the storage location internally.
For live code editing without rebuilding, use a **bind mount** instead:
```yaml
volumes:
  - ./app:/app
```
This maps your project folder directly into the container, so file edits on
your host machine appear inside the container immediately.

## Networking (Key Concept)

Inside a Compose stack, **each service name becomes a hostname** other
services can use to reach it. For example, if a `pgadmin` service needs to
reach a `postgres` service, it connects using the hostname `postgres` — not
`localhost`. Docker provides this DNS resolution automatically, since all
services in the same Compose file share a private virtual network.

## Common Errors Explained

**`address already in use`** — something on your host machine is already
using that port. Fix: change the host-side port in your `ports:` mapping
(e.g. `"5001:5000"` instead of `"5000:5000"`), or stop whatever else is using it.

**`failed to resolve host`** — the containers trying to talk to each other
aren't on the same Compose network, usually because they're defined in
separate `docker-compose.yml` files or one was started independently with
`docker run`. Fix: make sure both services are in the same Compose file and
restart with `docker compose down && docker compose up -d`.

## Mental Model Summary

- Image = frozen filesystem
- Container = running process
- Volume = persistent data
- Network = private virtual LAN
- Compose = multi-container orchestration

See `docker-flask-postgres-project.md` for a full worked example (Flask API +
PostgreSQL + Adminer) using everything above.