# Docker Setup — Ubuntu 24.04 LTS

Official install docs: https://docs.docker.com/engine/install/ubuntu/

## What Docker Actually Is

Docker consists of three major parts:
- **Docker daemon (`dockerd`)** — background service that manages containers
- **Docker CLI (`docker`)** — the command you type
- **containerd** — low-level runtime that launches Linux processes

Containers are normal Linux processes, isolated using kernel features, sharing
the host kernel. Docker does **not** virtualize hardware — it virtualizes the
filesystem and process tree.

## Install Docker (Official Method)

### 1. Remove conflicting packages
```bash
sudo apt remove -y docker docker-engine docker.io containerd runc
```
Removes Ubuntu's unofficial Docker packages, which conflict with the official ones.

### 2. Install prerequisites
```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg
```
These enable secure downloads and cryptographic verification of the packages.

### 3. Create keyring directory
```bash
sudo install -m 0755 -d /etc/apt/keyrings
```
`-m 0755` sets directory permissions (owner: read/write/execute, group and
others: read/execute). This is where APT stores trusted signing keys.

### 4. Add Docker's official GPG key
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

sudo chmod a+r /etc/apt/keyrings/docker.gpg
```
- `-fsSL` — fail silently on error, suppress progress output, follow redirects
- `gpg --dearmor` — converts the key to binary format APT expects
- This lets Ubuntu verify Docker packages are authentic before installing them

### 5. Add Docker repository
```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu noble stable" \
| sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
Tells `apt` where to find Docker packages. `$(dpkg --print-architecture)`
auto-detects your system architecture (e.g. `amd64`) so the right build is used.

### 6. Install Docker Engine + tools
```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
Installs the Docker daemon, CLI, container runtime, build system, and Docker
Compose v2 (the `docker compose` subcommand, not the legacy standalone
`docker-compose`).

### 7. Verify installation
```bash
docker --version
docker compose version
```

## Running Docker Without sudo

By default, Docker requires root access since it talks to a privileged socket.

```bash
sudo usermod -aG docker $USER
```
- `-aG docker` — appends (`-a`) your user to the `docker` group, so it can
  access the Docker socket without `sudo`
- **Log out and log back in** — group membership changes only apply at login,
  not mid-session

Verify it worked:
```bash
docker run --rm hello-world
```
If this runs without `sudo`, Docker is fully configured. `hello-world` is a
minimal test image that pulls, runs, prints a confirmation message, and exits.

## Manage the Docker Service

```bash
sudo systemctl status docker    # check if the docker daemon is running
sudo systemctl start docker     # start docker if it isn't running
sudo systemctl enable docker    # start docker automatically on boot (recommended for servers)
```
Docs: https://docs.docker.com/config/daemon/systemd/