# Git Basics

Docs: https://git-scm.com/docs

## Creating / Getting a Repository

```bash
cd my_project
git init                                          # create a new repo locally

git clone https://github.com/username/repo.git      # clone via HTTPS
git clone git@github.com:username/repo.git             # clone via SSH (see git-github-setup.md)
git clone repo.git my-folder-name                         # clone into a custom folder name
```

## Checking Status & Changes

```bash
git status                # what's changed, staged, or untracked
git diff                    # unstaged changes, line-by-line
git diff --staged             # staged changes, line-by-line (about to be committed)
git log                         # commit history
git log --oneline               # condensed, one line per commit
git log --oneline --graph --all   # visual branch history — useful once branching is involved
```

## Staging & Committing

```bash
git add script.py           # stage a single file
git add folder/                # stage everything in a folder
git add .                        # stage all changes in the current directory and below
git add -p                         # interactively stage specific chunks within files (review each hunk)

git commit -m "short description of change"
git commit -am "message"       # stage all TRACKED file changes + commit in one step (skips new/untracked files)
```
`git add -p` is worth knowing once commits should be focused — it lets you
split unrelated changes in the same file into separate, logical commits
instead of one commit doing several unrelated things.

### Writing Good Commit Messages
```
<type>: short summary (50 chars or less, imperative mood)

Optional longer explanation of WHY the change was made,
wrapped around 72 chars per line, if the summary alone
isn't enough context.
```
Imperative mood convention: "Add user login" not "Added user login" — think
of it as completing the sentence "This commit will ___".

## Pushing to GitHub

```bash
git push -u origin main       # first push: set the upstream tracking branch
git push                        # subsequent pushes, once upstream is set
```
- `-u` (`--set-upstream`) — links your local `main` branch to `origin/main`,
  so future `git push`/`git pull` don't need the branch name specified again

## `.gitignore`

Prevents specified files/patterns from ever being tracked or accidentally
committed — critical for secrets (`.env`), build artifacts, and OS files.

`.gitignore`:
```
# Environment / secrets
.env
*.pem

# Python
__pycache__/
*.pyc
venv/
.venv/

# Node
node_modules/

# OS
.DS_Store

# Editor
.vscode/
.idea/
```
Docs: https://git-scm.com/docs/gitignore

`.gitignore` only prevents *untracked* files from being added — if a file
was already committed before being added to `.gitignore`, it needs to be
explicitly untracked:
```bash
git rm --cached filename        # stop tracking, but keep the file on disk
git rm --cached -r folder/        # same, for a whole folder
```

## Viewing a Specific File's History

```bash
git log --follow filename          # history of a file, including through renames
git blame filename                    # who last changed each line, and in which commit
```

## Ignoring Already-Tracked File Changes Locally

```bash
git update-index --assume-unchanged filename
```
Useful for local-only config tweaks (e.g. a debug flag) you don't want to
accidentally commit, without adding the file to `.gitignore` for everyone.
Reverse with `--no-assume-unchanged`.