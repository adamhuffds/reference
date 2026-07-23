# Checking Mounted Drives in Linux

Reference notes for inspecting block devices, mountpoints, and disk usage on
Ubuntu/Debian-based systems.

## Quick Checks

### `df` — disk free space by mounted filesystem

```bash
df -h
```

- `-h` — "human-readable"; prints sizes in GB/MB instead of raw byte counts.
- Fastest way to see what's mounted, where, and current usage/free space.

Docs: <https://man7.org/linux/man-pages/man1/df.1.html>

### `lsblk` — list block devices

```bash
lsblk -f
```

- No flags gives a tree view of devices/partitions and their mountpoints.
- `-f` adds filesystem type (ext4, xfs, ntfs, etc.) and UUID — useful for
  identifying a drive before adding it to `/etc/fstab`.

Docs: <https://man7.org/linux/man-pages/man8/lsblk.8.html>

### `mount` — raw mount table

```bash
mount | grep "^/dev"
```

- `mount` with no arguments lists every mounted filesystem, including
  pseudo-filesystems (`proc`, `tmpfs`, `cgroup`, etc.), which makes the raw
  output noisy.
- `grep "^/dev"` filters to lines starting with `/dev`, keeping only actual
  physical/virtual devices.

Docs: <https://man7.org/linux/man-pages/man8/mount.8.html>

## More Detail

### `findmnt` — formatted, filterable mount tree

```bash
findmnt
findmnt /mnt/data
```

- With no arguments, prints all mounts as a readable tree (nicer than raw
  `mount` output).
- Passing a path checks whether that specific mountpoint exists and shows its
  details — handy for confirming a drive mounted where expected.

Docs: <https://man7.org/linux/man-pages/man8/findmnt.8.html>

### `/etc/fstab` — configured (boot-time) mounts

```bash
cat /etc/fstab
```

- Shows what *should* mount at boot, as opposed to `df`/`mount`/`lsblk`,
  which show what's *currently* mounted.
- Useful for diagnosing a drive that failed to auto-mount by comparing
  intended vs. actual state.

Docs: <https://man7.org/linux/man-pages/man5/fstab.5.html>

## Combined Check

```bash
lsblk -f && echo "---" && df -h
```

- `&&` chains commands so the second runs only if the first succeeds.
- `echo "---"` prints a separator so the two outputs are visually distinct.
- Gives physical device/partition layout with filesystem types (`lsblk -f`)
  alongside actual usage and mountpoints (`df -h`) in one pass.

## Summary Table

| Command | Shows | Best for |
|---|---|---|
| `df -h` | Mounted filesystems, usage/free space | Quick space check |
| `lsblk -f` | Block device tree, mountpoints, filesystem type | Identifying drives/partitions |
| `mount \| grep "^/dev"` | Raw mount table, device options | Checking exact mount options |
| `findmnt <path>` | Formatted tree, single-mountpoint lookup | Verifying a specific mount |
| `cat /etc/fstab` | Configured boot-time mounts | Comparing intended vs. actual state |