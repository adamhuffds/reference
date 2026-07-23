# SSH Setup: Laptop to Home Server (Local LAN)

A step-by-step guide for setting up SSH key-based authentication from a laptop to a home server on a local network. Covers key generation, config file setup, and common troubleshooting scenarios.

**Environment:** Ubuntu 24.04 (both machines)  
**Auth method:** Ed25519 key pair (password auth disabled on server)

---

## Prerequisites

- SSH server already running on the home server (`openssh-server`)
- SSH client installed on laptop (`ssh -V` to verify)
- You know the server's local IP address (`ip a` on the server, or `arp -a` on the laptop)
- At least one way to access the server initially (physical access, another machine with an existing key, USB, etc.)

---

## Step 1: Generate a New Key Pair on Your Laptop

Generate a dedicated key pair for this machine. Avoid reusing keys across machines or services (e.g. GitHub) — separate keys mean independent revocation and clear auditability.

```bash
ssh-keygen -t ed25519 -C "your-laptop-label" -f ~/.ssh/id_ed25519_homeserver
```

| Flag | Purpose |
|------|---------|
| `-t ed25519` | Key type — Ed25519 is modern, fast, and more secure than RSA |
| `-C "your-laptop-label"` | Comment embedded in the public key for identification |
| `-f ~/.ssh/id_ed25519_homeserver` | Explicit filename — avoids colliding with other keys (e.g. GitHub) |

Set a passphrase when prompted — it encrypts the private key on disk.

This creates two files:
- `~/.ssh/id_ed25519_homeserver` — private key, never leaves your laptop
- `~/.ssh/id_ed25519_homeserver.pub` — public key, copied to the server

---

## Step 2: Copy the Public Key to the Server

### Option A: ssh-copy-id (if password auth is enabled)

```bash
ssh-copy-id -i ~/.ssh/id_ed25519_homeserver.pub user@192.168.x.x
```

This appends your public key to `~/.ssh/authorized_keys` on the server automatically.

### Option B: Manual copy (if password auth is disabled)

If the server only accepts key auth, you need an existing way in — another machine with a working key, physical access, USB, etc.

**Print the public key on your laptop:**

```bash
cat ~/.ssh/id_ed25519_homeserver.pub
```

Copy the entire line (including the `ssh-ed25519` prefix and the comment at the end).

**On the server, append it to the correct user's `authorized_keys`:**

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
echo "paste-full-public-key-here" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

> **Note:** The public key is safe to transmit via email, USB, or any other method. Only the private key must be kept secure and never shared.

**If you have access via another machine**, you can use `grep` to copy a specific key by its comment rather than retyping it:

```bash
grep "your-laptop-label" /home/otheruser/.ssh/authorized_keys >> /home/targetuser/.ssh/authorized_keys
```

---

## Step 3: Verify the Key Landed Correctly

On the server:

```bash
# Check the key is present and on a single unbroken line
cat ~/.ssh/authorized_keys

# Check for hidden characters or line breaks
cat -A ~/.ssh/authorized_keys

# Count lines — each key should be exactly one line
wc -l ~/.ssh/authorized_keys
```

A correct entry looks like:
```
ssh-ed25519 AAAAC3Nza...rest of key... your-laptop-label$
```

The `$` at the end (shown by `cat -A`) means a clean Unix line ending. If you see `^M$`, Windows-style line endings snuck in and will break auth.

---

## Step 4: Check Permissions on the Server

SSH silently rejects auth if permissions are wrong. Always verify:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

Expected output from `ls -la ~/.ssh/`:
```
drwx------  .ssh
-rw-------  authorized_keys
```

---

## Step 5: Configure ~/.ssh/config on Your Laptop

The config file lets you connect with a short alias instead of typing the full IP and username each time. It also explicitly maps which key to use for which host.

```bash
touch ~/.ssh/config
chmod 600 ~/.ssh/config
```

Open `~/.ssh/config` and add entries for both your server and any other services (e.g. GitHub) to avoid ambiguity:

```
# GitHub
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519

# Home Server
Host homeserver
    HostName 192.168.x.x
    User your-server-username
    IdentityFile ~/.ssh/id_ed25519_homeserver
    ServerAliveInterval 60
    ServerAliveCountMax 3
```

| Option | Purpose |
|--------|---------|
| `Host` | Alias used on the command line (`ssh homeserver`) |
| `HostName` | Actual IP or hostname SSH connects to |
| `User` | Username on the remote machine |
| `IdentityFile` | Explicit private key to use — no guessing |
| `ServerAliveInterval 60` | Sends a keepalive packet every 60 seconds |
| `ServerAliveCountMax 3` | Drops connection after 3 missed keepalives |

> **Important:** The `User` field must match the actual username on the server. A mismatch here will cause `Permission denied (publickey)` even if the key is correctly set up.

---

## Step 6: Test the Connection

```bash
ssh homeserver
```

Test your GitHub key still works (optional):

```bash
ssh -T git@github.com
```

---

## Step 7: Add Key to ssh-agent (Optional)

If you set a passphrase, `ssh-agent` caches it for your session so you only type it once:

```bash
eval "$(ssh-agent -s)"        # start the agent
ssh-add ~/.ssh/id_ed25519_homeserver  # register the key
ssh-add -l                    # list all loaded keys to confirm
```

---

## Troubleshooting

### Connection hangs (no output, cursor on next line)

SSH is reaching out but getting no response. Verify the IP is correct and reachable:

```bash
ping 192.168.x.x
```

Find the server's actual IP on the server:

```bash
ip a
```

### Permission denied (publickey)

Run verbose mode to see exactly what SSH is attempting:

```bash
ssh -vv homeserver
```

Key things to look for:
- `Offering public key: ~/.ssh/id_ed25519_homeserver` — confirms the right key is being tried
- The username in the final `Permission denied` line — a wrong username is a common cause
- `Server rejected key` vs `Server accepts key`

Check the server's effective SSH config:

```bash
sudo sshd -T | grep -E "pubkeyauthentication|authorizedkeysfile|passwordauthentication"
```

The `authorizedkeysfile` value tells you where SSH is actually looking for keys — it may not be `~/.ssh/authorized_keys` on all systems.

### Key is correct but auth still fails

Common causes:
- **Wrong user** — the key is in `/home/alice/.ssh/authorized_keys` but you're connecting as `bob`
- **Permissions too open** — fix with `chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys`
- **Key split across multiple lines** — check with `cat -A ~/.ssh/authorized_keys`
- **Windows line endings** — look for `^M$` in `cat -A` output

---

## Future: Adding Remote Access

When you're ready to access the server from outside your LAN:

1. **Port forwarding** — forward an external port (22 or a custom port) to your server's local IP on your router
2. **Dynamic DNS** — if your ISP assigns a dynamic public IP, a service like [DuckDNS](https://www.duckdns.org/) gives you a stable hostname
3. **Harden sshd config** — consider changing the default port, restricting which users can connect, and setting `MaxAuthTries`

The key pair you created in this guide is already usable for remote access — no changes needed on the key side.

---

## Reference

- [OpenSSH man page — ssh](https://man.openbsd.org/ssh)
- [OpenSSH man page — ssh-keygen](https://man.openbsd.org/ssh-keygen)
- [OpenSSH man page — ssh_config](https://man.openbsd.org/ssh_config)
- [OpenSSH man page — sshd_config](https://man.openbsd.org/sshd_config)
- [Ubuntu OpenSSH Server docs](https://ubuntu.com/server/docs/openssh-server)
- [Arch Wiki — SSH keys](https://wiki.archlinux.org/title/SSH_keys) (excellent reference even on Ubuntu)