# Git — Remotes & GitHub Workflow

Docs: https://docs.github.com/en/get-started

## Remotes

A "remote" is a named reference to another copy of the repository —
typically GitHub. `origin` is the conventional name for the primary remote.

```bash
git remote -v                                  # list remotes and their URLs
git remote add origin git@github.com:user/repo.git   # add a remote (after 'git init', before first push)
git remote set-url origin <new-url>                    # change an existing remote's URL
git remote remove origin                                  # remove a remote
```

## Fetch vs. Pull

```bash
git fetch origin            # download remote changes, but DON'T merge into your local branch
git pull origin main           # fetch AND merge (or rebase, depending on config) in one step
```
`fetch` is the safer choice when you want to see what changed before
deciding how to integrate it — `git log origin/main` or `git diff
main origin/main` after a fetch shows you what's new without touching your
working files.

```bash
git config pull.rebase false     # 'git pull' merges (default, creates a merge commit if diverged)
git config pull.rebase true        # 'git pull' rebases instead (cleaner history, see git-branching.md)
```

## Typical Feature Branch Workflow

```bash
git switch main
git pull                                    # make sure main is current
git switch -c feature/add-login               # branch off main

# ... make changes, commit ...

git push -u origin feature/add-login              # push the new branch to GitHub
```
Then open a Pull Request on GitHub's web UI (or via CLI, see below) to merge
`feature/add-login` into `main` once it's reviewed.

## Pull Requests (PRs)

A PR is GitHub's mechanism for proposing that one branch's changes be merged
into another, with room for code review, comments, and CI checks before
merging — the standard collaboration unit on GitHub, even for solo projects
where it just serves as a review checkpoint against your own work.

### Via GitHub CLI (`gh`)
```bash
sudo apt install gh                    # if not already installed
gh auth login                             # one-time authentication setup

gh pr create --title "Add login" --body "Implements user login via Flask-Login"
gh pr list                                  # list open PRs
gh pr view 12                                 # view PR #12
gh pr checkout 12                               # check out someone else's PR branch locally to test it
gh pr merge 12                                    # merge a PR from the terminal
```
Docs: https://cli.github.com/manual/

### Keeping a Feature Branch Updated (While a PR is Open)
```bash
git switch feature/add-login
git fetch origin
git merge origin/main          # or: git rebase origin/main
```
Pull in the latest `main` periodically on long-lived feature branches to
avoid a huge conflict-resolution session at merge time.

## Forking (Contributing to Repos You Don't Own)

```bash
gh repo fork owner/repo --clone            # fork on GitHub + clone it locally in one step

git remote add upstream git@github.com:owner/repo.git   # track the original repo as 'upstream'
git fetch upstream
git merge upstream/main                                    # pull the original repo's latest changes into your fork
```
Standard open-source contribution flow: fork → branch → commit → push to
your fork → open a PR from your fork's branch into the original repo.

## Cloning Someone Else's Repo Read-Only

```bash
git clone https://github.com/user/repo.git      # fine for read-only, no push access needed
```
HTTPS is sufficient (no SSH key setup required) if you're only ever pulling,
not pushing, to a given repo.

## GitHub Issues (Linking Commits/PRs to Issues)

```bash
git commit -m "Fix login redirect bug

Closes #42"
```
Including `Closes #42`, `Fixes #42`, or `Resolves #42` in a commit or PR
description automatically closes that issue when the commit/PR merges into
the default branch — a convention worth using consistently for traceability.
Docs: https://docs.github.com/en/issues/tracking-your-work-with-issues/linking-a-pull-request-to-an-issue

## GitHub Actions (Brief Mention)

CI/CD workflows defined in `.github/workflows/*.yml`, triggered on events
like `push` or `pull_request` — e.g. automatically running tests on every
PR. Deep enough a topic to warrant its own reference doc later if/when
you're setting up CI for a project.