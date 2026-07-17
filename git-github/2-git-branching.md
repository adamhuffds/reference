# Git — Branching, Merging & Rebasing

Docs: https://git-scm.com/docs/git-branch

## Why Branch

A branch is an independent line of development — the standard workflow is
to never commit directly to `main` for real changes, instead creating a
branch per feature/fix, then merging it back once it's done and reviewed.

## Creating & Switching Branches

```bash
git branch                        # list local branches, * marks the current one
git branch -a                       # list local AND remote-tracking branches
git branch new-feature                 # create a new branch (doesn't switch to it)

git switch new-feature                    # switch to an existing branch (modern command)
git switch -c new-feature                   # create AND switch in one step (modern)

git checkout new-feature                      # older equivalent to 'switch'
git checkout -b new-feature                     # older equivalent to 'switch -c'
```
`git switch` (Git 2.23+) is the current recommended command for changing
branches — `git checkout` historically did double duty for both switching
branches *and* restoring files, which was a frequent source of mistakes.
`switch` (and `restore`, for file-level changes) split that ambiguous
command into two clearer ones. Older tutorials/muscle memory will still use
`checkout` — both work, but prefer `switch`/`restore` going forward.

## Renaming & Deleting Branches

```bash
git branch -m old-name new-name       # rename a branch
git branch -d feature-branch             # delete a branch (safe — refuses if unmerged changes exist)
git branch -D feature-branch               # force delete, even if unmerged
```

## Merging

```bash
git switch main               # switch to the branch you want to merge INTO
git merge new-feature            # bring new-feature's changes into main
```
Two possible outcomes:
- **Fast-forward** — if `main` hasn't changed since the branch was created,
  Git just moves the pointer forward, no merge commit needed
- **Merge commit** — if both branches have diverged, Git creates a new
  commit joining the two histories together

```bash
git merge --no-ff new-feature
```
Forces a merge commit even when a fast-forward would be possible — some
teams prefer this since it preserves a visible record that a feature branch
existed, rather than flattening its history into `main`'s linear log.

## Merge Conflicts

Happen when the same lines were changed differently on both branches. Git
marks the conflict directly in the file:
```
<<<<<<< HEAD
this is main's version
=======
this is new-feature's version
>>>>>>> new-feature
```
Resolve by manually editing the file to the correct final content (removing
the `<<<<<<<`/`=======`/`>>>>>>>` markers), then:
```bash
git add resolved-file.py
git commit                # completes the merge
```
`git merge --abort` — bail out of a conflicted merge entirely, returning to
the state before `git merge` was run, if you'd rather restart.

## Rebasing (Alternative to Merging)

```bash
git switch new-feature
git rebase main
```
Instead of creating a merge commit, rebase replays your branch's commits on
top of the latest `main`, producing a clean, linear history. Conflicts
during a rebase are resolved the same way as a merge, one commit at a time:
```bash
# after resolving conflicts in a file:
git add resolved-file.py
git rebase --continue
# or, to bail out entirely:
git rebase --abort
```

### Merge vs. Rebase — When to Use Which
- **Merge** — preserves exact history, safe on shared/public branches,
  slightly messier log with merge commits
- **Rebase** — cleaner, linear history, but **rewrites commit hashes** —
  never rebase a branch that others have already pulled/based work on,
  since it breaks their history alignment. Safe on your own local,
  not-yet-pushed feature branches.

## Interactive Rebase (Cleaning Up Commit History)

```bash
git rebase -i HEAD~3        # interactively edit the last 3 commits
```
Opens an editor listing recent commits with an action per line (`pick`,
`squash`, `reword`, `drop`, etc.) — commonly used to squash several small
"wip" commits into one clean commit before merging/pushing a feature branch.
Docs: https://git-scm.com/docs/git-rebase#_interactive_mode

## Cherry-Picking

```bash
git cherry-pick <commit-hash>
```
Applies a single specific commit from another branch onto your current
branch, without merging the whole branch — useful for pulling one bugfix
into a release branch without bringing along unrelated in-progress work.

## Tags (Marking Releases)

```bash
git tag v1.0.0                    # lightweight tag on the current commit
git tag -a v1.0.0 -m "Release 1.0.0"   # annotated tag (recommended — stores author, date, message)
git push origin v1.0.0                   # tags aren't pushed automatically — must push explicitly
git push origin --tags                     # push all tags at once
```