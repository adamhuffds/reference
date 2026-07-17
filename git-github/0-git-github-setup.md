# Git & GitHub — Install, Configure & SSH Setup

Docs: https://git-scm.com/doc

## Install

```bash
sudo apt update
sudo apt install git
git --version
```

## Configure Your Identity

```bash
git config --global user.name "Your Name"
git config --global user.email "you@email.com"
```
- `--global` — applies to every repository for your Linux user, stored in
  `~/.gitconfig`. Omit `--global` inside a specific repo to override it for
  that project only (e.g. a different email for work vs. personal repos).
- This name/email is attached to every commit you make — GitHub matches the
  email against your account to attribute commits to your profile.

```bash
git config --list                # show all current config values
git config --list --show-origin    # also show which file each setting came from
```

## Useful Global Config Defaults

```bash
git config --global init.defaultBranch main       # new repos start on 'main' instead of legacy 'master'
git config --global core.editor "nano"              # or "vim", "code --wait" for VS Code
git config --global pull.rebase false                  # merge (not rebase) by default on 'git pull' — see git-branching.md
git config --global color.ui auto                        # colored output in the terminal
```

## SSH Key Setup for GitHub (Recommended over HTTPS + password)

GitHub no longer accepts password authentication over HTTPS for git
operations — you need either an SSH key or a Personal Access Token. SSH is
the more common setup for a personal dev machine since you authenticate
once and never re-enter credentials.

### 1. Check for an existing key
```bash
ls -al ~/.ssh
```
Look for `id_ed25519.pub` or `id_rsa.pub` — if one exists, you can reuse it
(skip to step 3) or generate a new one.

### 2. Generate a new SSH key
```bash
ssh-keygen -t ed25519 -C "you@email.com"
```
- `-t ed25519` — key type; Ed25519 is the current recommended algorithm
  (faster and more secure than the older `rsa` type still seen in a lot of
  tutorials)
- `-C` — a comment (typically your email) to help identify the key later
- Press Enter to accept the default file location, and optionally set a
  passphrase for extra security (prompted at each first use per session if set)

### 3. Add the key to the SSH agent
```bash
eval "$(ssh-agent -s)"          # start the agent
ssh-add ~/.ssh/id_ed25519        # add your key to it
```

### 4. Copy the public key
```bash
cat ~/.ssh/id_ed25519.pub
```
Copy the full output (starts with `ssh-ed25519`).

### 5. Add it to GitHub
GitHub → Settings → SSH and GPG keys → New SSH key → paste the public key.
Docs: https://docs.github.com/en/authentication/connecting-to-github-with-ssh

### 6. Verify the connection
```bash
ssh -T git@github.com
```
Should respond with `Hi username! You've successfully authenticated...`

### 7. Use SSH-style remote URLs
```bash
git clone git@github.com:username/repo.git
```
Note the `git@github.com:` format — not `https://github.com/...`. If a repo
was already cloned via HTTPS, switch its remote:
```bash
git remote set-url origin git@github.com:username/repo.git
```

## GPG Commit Signing (Optional, for Verified Commits)

Adds a "Verified" badge on GitHub, cryptographically proving a commit
actually came from you.
```bash
gpg --full-generate-key                  # generate a GPG key if you don't have one
gpg --list-secret-keys --keyid-format=long
git config --global user.signingkey <KEY_ID>
git config --global commit.gpgsign true
```
Add the public GPG key to GitHub → Settings → SSH and GPG keys, separately
from your SSH key. Docs: https://docs.github.com/en/authentication/managing-commit-signature-verification