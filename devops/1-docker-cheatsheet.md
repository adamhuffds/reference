# Docker Cheat Sheet

Full CLI reference: https://docs.docker.com/reference/cli/docker/

## Mental Model

- **Image** — frozen filesystem template (like a class)
- **Container** — running instance of an image (like an object)
- **Volume** — persistent storage that survives container removal
- **Network** — private virtual LAN connecting containers

Containers are disposable. Images are immutable until rebuilt.

## Images

```bash
docker images                    # list images stored locally
docker pull python:3.12          # download an image from Docker Hub
docker rmi image_name             # remove an image
docker image prune                # remove unused (dangling) images
```

## Running Containers

```bash
docker run hello-world                          # good sanity test that docker works
docker run python:3.12                          # run a container from an image
docker run -it python:3.12                       # run interactively (see flags below)
docker run --rm -it python:3.12                  # run interactively, auto-delete on exit
docker run --name my_python -it python:3.12       # run with a specific container name
```
- `-i` — keep STDIN open (interactive input)
- `-t` — allocate a pseudo-terminal
- `-it` together = a usable interactive shell session
- `--rm` — automatically remove the container when it exits (prevents clutter
  from accumulating stopped containers)
- `--name` — assign a human-readable name instead of a random one, so you can
  reference it later (`docker start my_python`) instead of copying container IDs

## Starting / Stopping

```bash
docker start my_python           # start an existing (stopped) container
docker stop my_python            # gracefully stop a running container
docker stop container_id          # same, referenced by ID instead of name
```

## Listing & Inspecting

```bash
docker ps                        # list running containers
docker ps -a                     # list all containers, including stopped ones
```

## Removing

```bash
docker rm container_id            # remove a specific (stopped) container
docker rm $(docker ps -aq)        # remove all stopped containers at once

docker container prune            # remove all stopped/inactive containers
docker image prune                # remove unused images
docker system prune               # remove all unused containers, images, networks (careful — broad cleanup)
```
`$(docker ps -aq)` — `-a` lists all containers, `-q` outputs only their IDs
(quiet mode), so the command substitution feeds every container ID into `docker rm`.

## Volumes (Persisting Data)

Containers are disposable by design — anything written inside one is lost
when it's removed unless stored in a volume.

```bash
docker volume create pgdata                 # create a named volume, managed by Docker internally

# Run with a bind mount (host folder <-> container folder)
docker run -it \
  -v ~/docker-share:/app \
  python:3.12
```
- `-v host_path:container_path` — maps a folder on your real machine
  (`~/docker-share`) to a folder inside the container (`/app`), so files
  persist and are editable from either side

## Ports (Needed for Web Apps)

```bash
docker run -p 5000:5000 flask-app
```
- `-p host_port:container_port` — forwards traffic from a port on your host
  machine to a port inside the container, so a browser hitting
  `localhost:5000` reaches the app running inside the container

## Passing Command-Line Arguments

```bash
docker run python-test one two three
```
Anything after the image name is passed as arguments to the container's
default command (`CMD` in the Dockerfile) — commonly used in ETL pipelines to
parameterize a job at runtime.

## Entering a Running Container's Shell

```bash
docker run -it python-test bash
```
Drops you into an interactive `bash` shell inside the container, useful for
poking around the filesystem or testing commands before scripting them.