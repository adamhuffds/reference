# fdupes & jdupes — Duplicate File Finder Reference

fdupes: https://github.com/adrianlopezroche/fdupes
jdupes: https://github.com/jbruchon/jdupes

Both tools find duplicate files by comparing file size first, then a hash of
the contents (not just filename) — so renamed copies of the same file are
still correctly detected as duplicates.

## fdupes vs. jdupes — Which to Use

**jdupes** is a faster, actively maintained fork of fdupes with additional
features (hardlinking, better performance on large file sets, more output
options). Unless you have a specific reason to use the original, **jdupes
is the current recommended choice** — fdupes development has been much
slower in comparison. The command syntax below is largely interchangeable
between the two; differences are called out where they exist.

## Install

```bash
# fdupes
sudo apt install fdupes

# jdupes (may need to build from source or grab a release binary,
# depending on your Ubuntu version's repo availability)
sudo apt install jdupes
```
If `jdupes` isn't in your distro's repos, check release binaries directly:
https://github.com/jbruchon/jdupes/releases

## Basic Usage

```bash
fdupes folder                  # find duplicates in a single folder (not recursive)
fdupes -r folder                 # find duplicates recursively (subfolders included)
fdupes -r folderA folderB          # find duplicates ACROSS two different folders
```
Output groups duplicate files together, separated by blank lines — the
first file listed in each group is not inherently "the original," just the
first one encountered during the scan.

## Interactive Deletion

```bash
fdupes -rd folder
```
- `-d` — prompt interactively for each duplicate group, asking which
  file(s) to keep and which to delete
- Always review the prompt carefully — this is destructive once confirmed

```bash
fdupes -rdN folder
```
- `-N` — combined with `-d`, automatically preserves the **first** file in
  each group and deletes the rest, without per-file prompts. Still shows
  what's being deleted before/as it happens, but with far less interaction
  than plain `-d`.

## Dry Run First (Strongly Recommended)

Before deleting anything, always inspect what's actually flagged as a
duplicate:
```bash
fdupes -r folder > duplicates.txt        # save the list to review first
```
There is no single "dry run" flag for the deletion itself — the safe
pattern is to run a plain (non-`-d`) scan first, review the output, *then*
run the deletion pass once you're confident about what will be removed.

## Useful Flags (fdupes)

```bash
fdupes -r -S folder            # show file size next to each duplicate
fdupes -r -m folder              # summarize: just show counts/space that could be reclaimed, no file list
fdupes -r -1 folder                # print each duplicate set on a single line (easier to script/parse)
fdupes -r -A folder                  # exclude hidden files/directories from the scan
fdupes -r -n folder                    # exclude zero-length (empty) files from being considered duplicates
```

## jdupes-Specific Features

### Hardlinking Instead of Deleting
```bash
jdupes -r -L folder
```
Replaces duplicate files with hardlinks to a single copy on disk — this
reclaims disk space **without actually deleting any files**, since all the
hardlinked paths still resolve to real, readable files. Safer than deletion
for cases where you want multiple accessible copies/paths but don't need
multiple copies taking up disk space.
Note: hardlinks only work within the same filesystem/partition — this won't
work across separate drives or mount points.

### Symlinking Instead of Deleting
```bash
jdupes -r -l folder
```
Similar idea, but uses symlinks instead of hardlinks — works across
filesystems, but breaks if the original target file is later moved/deleted
(hardlinks don't have this issue, since they point directly to the same
underlying data, not a path).

### Performance Options
```bash
jdupes -r -O folder          # order files by modification time before comparing (can change which file is "first")
jdupes -r -Q folder            # quick mode — compares file hashes only, not full byte-for-byte confirmation
                                  # (very slightly higher false-positive risk on hash collisions, but much faster on huge datasets)
```

## Finding Duplicates Between Two Folders (Adam's Common Use Case)

```bash
fdupes -r path/to/folderA path/to/folderB
```
When scanning across two folders, `fdupes`/`jdupes` still just reports
duplicate groups — it doesn't inherently know which folder should be
considered the "source of truth." Review output carefully before deleting;
consider deleting only from one specific folder deliberately:
```bash
fdupes -r folderA folderB | grep "^path/to/folderB"
```
This filters the duplicate list to only show matches living in `folderB`,
letting you build a targeted deletion list rather than trusting `-N`'s
"keep the first file" ordering across two folders you care about
differently.

## Safety Checklist Before Running a Deletion Pass

- [ ] Run a plain (non-deleting) scan first and actually read the output
- [ ] Confirm you have a backup, or use `jdupes -L` (hardlink) instead of
      deletion if you're not fully confident
- [ ] Be extra cautious with `-N` across multiple folders — "first file
      encountered" may not be the copy you actually want to keep
- [ ] Avoid running against system directories or anything outside a
      folder you explicitly intend to deduplicate