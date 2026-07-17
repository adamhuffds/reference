# Git — Undoing Changes

Docs: https://git-scm.com/book/en/v2/Git-Basics-Undoing-Things

## Discarding Uncommitted Changes

```bash
git restore filename            # discard unstaged changes to a file, revert to last commit's version
git restore --staged filename     # unstage a file (keeps the changes, just removes from staging)
git restore .                       # discard ALL unstaged changes in the current directory
git clean -n                          # preview what untracked files WOULD be deleted (dry run)
git clean -f                            # actually delete untracked files (careful — not recoverable via git)
git clean -fd                             # also remove untracked directories
```
`git restore` is the modern command for file-level undo — it replaced the
file-restoring half of `git checkout`'s historically dual-purpose role (see
`git-branching.md` for the branch-switching half, now `git switch`).

## Amending the Last Commit

```bash
git commit --amend -m "corrected message"       # change the last commit's message
git add forgotten_file.py
git commit --amend --no-edit                       # add a forgotten file to the last commit, keep the same message
```
Only amend commits that **haven't been pushed yet** — amending rewrites the
commit hash, which causes problems for anyone who already pulled the
original version (same caution as rebasing, see `git-branching.md`).

## Reset (Moving the Branch Pointer)

```bash
git reset --soft HEAD~1      # undo the last commit, keep changes staged
git reset HEAD~1               # undo the last commit, keep changes unstaged (default: --mixed)
git reset --hard HEAD~1          # undo the last commit AND discard all changes — destructive, be sure first
```
- `--soft` — only moves the commit pointer; your files and staging area are
  untouched, so the changes are still there, just "un-committed"
- `--mixed` (default) — moves the pointer and unstages changes, but keeps
  the file contents on disk
- `--hard` — moves the pointer and discards changes entirely, matching your
  working directory to the target commit exactly

`git reset --hard` is the one command on this page that can genuinely lose
work if used carelessly — always double-check `git status`/`git diff`
before running it, or make sure the commit you're resetting past is
recoverable via `git reflog` (see below) if something goes wrong.

## Reset vs. Revert

```bash
git revert <commit-hash>
```
`revert` creates a **new commit** that undoes a previous commit's changes,
rather than rewriting history like `reset` does. This is the safe choice
for undoing something that's **already been pushed and possibly pulled by
others** — it doesn't rewrite shared history, so it won't break anyone
else's clone. Use `reset` only on local, not-yet-pushed commits.

## Stashing (Temporarily Shelve Changes)

```bash
git stash                          # save uncommitted changes, revert working directory to last commit
git stash save "wip: login form"     # stash with a descriptive message
git stash list                         # see all stashes
git stash pop                            # reapply the most recent stash AND remove it from the stash list
git stash apply                            # reapply the most recent stash but KEEP it in the list
git stash apply stash@{2}                    # reapply a specific stash by index
git stash drop stash@{2}                       # delete a specific stash without applying it
git stash clear                                  # delete all stashes
```
Common use case: you're mid-change on one branch and need to quickly switch
to fix something urgent on another — stash the in-progress work, switch
branches, fix the issue, switch back, `git stash pop` to resume exactly
where you left off.

```bash
git stash -u          # also stash untracked (new) files, not just modified tracked ones
```

## Reflog (Recovering "Lost" Commits)

```bash
git reflog
```
Git keeps a log of every place `HEAD` has pointed, including commits that
are no longer reachable from any branch (e.g. after a `reset --hard` or a
rebase gone wrong). This is the safety net for most "I think I deleted my
work" panics:
```bash
git reflog                        # find the commit hash from before the mistake
git reset --hard <commit-hash>       # or: git cherry-pick <commit-hash> to recover just that commit
```
Reflog entries expire eventually (default 90 days for reachable commits, 30
for unreachable ones) — not a permanent backup, but generally long enough
to recover from a recent mistake.

## Undoing a Merge

```bash
git reset --hard HEAD~1        # if the merge commit hasn't been pushed yet
git revert -m 1 <merge-commit-hash>   # if it HAS been pushed — creates a new commit undoing the merge
```
`-m 1` tells `revert` which parent of the merge commit to treat as the
"mainline" to revert back to — required specifically for merge commits,
since they have two parents instead of one.