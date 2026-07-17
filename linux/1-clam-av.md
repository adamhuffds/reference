# ClamAV (Antivirus Scanner)

ClamAV is an open-source antivirus engine, commonly used on Linux for
on-demand scanning (it doesn't run continuous real-time protection like a
typical Windows AV, unless paired with `clamav-daemon`/`clamonacc`).

Official docs: https://docs.clamav.net/

## Install

```bash
sudo apt update
sudo apt install clamav clamav-daemon
```
- `clamav` — the core scanning engine and `clamscan` command-line tool
- `clamav-daemon` — runs ClamAV as a background service (`clamd`), which
  allows faster repeated scans since the virus database stays loaded in
  memory instead of being reloaded on every scan

Docs: https://docs.clamav.net/manual/Installing.html

## Update Virus Definitions

```bash
sudo freshclam
```
- Downloads the latest virus signature database from ClamAV's servers.
- Run this before scanning if it's been a while — definitions go stale fast,
  and `clamav-daemon` normally handles this update automatically in the
  background once installed.

Docs: https://docs.clamav.net/manual/Usage/SignatureManagement.html

## Run a Scan

```bash
# General format
sudo clamscan target      # sudo likely only needed for system directories you don't own

# Scan a specific directory
sudo clamscan path
sudo clamscan path -r     # -r = recursive, scan all subdirectories too

# Scan the entire filesystem
sudo clamscan -r /
```
- `-r` — recurse into subdirectories (without it, `clamscan` only scans files
  directly in the given path, not nested folders)
- Scanning `/` recursively can take a long time depending on disk size — worth
  running in the background or via `nohup`/`tmux` for large scans.

Docs: https://docs.clamav.net/manual/Usage/Scanning.html

## Help / Support

```bash
sudo clamscan --help
```
- Prints all available `clamscan` flags and options directly from the tool.

Full command reference: https://docs.clamav.net/manual/Usage.html