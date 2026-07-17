# tmux Cheat Sheet

Docs: https://github.com/tmux/tmux/wiki

tmux is a terminal multiplexer — it lets you run terminal sessions that
persist independently of any single terminal window or SSH connection. You
can detach from a session (or lose your connection entirely) and reattach
later with everything — running processes, scrollback, pane layout — intact.

## Install

```bash
sudo apt update
sudo apt install tmux
tmux -V          # verify install, show version
```

## The Prefix Key

Nearly every tmux keyboard shortcut starts with a **prefix key**, default
`Ctrl+b`. You press the prefix, release it, then press the command key.
Shorthand in this doc: `PREFIX d` means "press Ctrl+b, release, then press d."

## Sessions

A session is a persistent tmux instance that can contain multiple windows.

```bash
tmux                          # start a new, unnamed session
tmux new -s mysession           # start a new NAMED session — naming is strongly recommended
tmux ls                            # list all running sessions
tmux attach -t mysession             # reattach to a named session
tmux attach                            # reattach to the most recent session
tmux kill-session -t mysession           # kill a specific session
tmux kill-server                           # kill ALL tmux sessions
```

### From inside a session
```
PREFIX d          # detach — session keeps running in the background
PREFIX $           # rename the current session
```
Naming sessions matters once you have more than one running — `tmux ls`
without names just shows numbered indexes that aren't meaningful at a glance.

## Windows (Tabs)

A window is like a browser tab within a session — one visible layout at a time.

```
PREFIX c          # create a new window
PREFIX ,           # rename the current window
PREFIX n            # next window
PREFIX p             # previous window
PREFIX 0-9             # jump directly to window number 0-9
PREFIX w                 # list all windows, pick one interactively
PREFIX &                   # kill the current window (asks for confirmation)
```

## Panes (Splits)

A pane is a subdivision within a window — for viewing multiple terminals
side by side (e.g. a log tail next to an active shell).

```
PREFIX %          # split pane vertically (side by side)
PREFIX "           # split pane horizontally (stacked)

PREFIX arrow-key     # move between panes (up/down/left/right)
PREFIX o               # cycle to the next pane
PREFIX q                 # briefly show pane numbers, then press a number to jump directly to it
PREFIX z                   # zoom the current pane to fullscreen (toggle) — great for focusing without losing the layout
PREFIX x                     # kill the current pane (asks for confirmation)

PREFIX PREFIX-arrow            # (hold) resize the current pane in that direction — or drag with mouse if enabled
```

### Even Out / Rearrange Pane Layouts
```
PREFIX space          # cycle through preset layouts (even-horizontal, even-vertical, main-vertical, etc.)
```

## Scrolling & Copy Mode

By default, your mouse wheel/PageUp won't scroll a tmux pane like a normal
terminal — you enter "copy mode" to scroll and select text.

```
PREFIX [             # enter copy mode (scroll with arrow keys or PageUp/PageDown)
q                       # exit copy mode
```
Inside copy mode (vi-style keybindings, if configured — see below):
```
space          # start text selection
enter            # copy selection, exit copy mode
```
```
PREFIX ]          # paste the most recently copied text
```

## Detaching & Reattaching — The Core Use Case

```bash
# Start a long-running task in tmux
tmux new -s upload
python long_running_script.py
# PREFIX d to detach — script keeps running

# Close your laptop, disconnect SSH, whatever — the process is unaffected

# Later, from the same machine (or reconnect via SSH):
tmux attach -t upload
```
This is the main reason to reach for tmux over a plain terminal — anything
you'd normally lose by closing a terminal window (or an SSH session
dropping) keeps running, and you can pick the exact terminal state back up.

## Useful Session/Pane Info Commands

```bash
tmux list-sessions              # same as 'tmux ls'
tmux list-windows -t mysession    # list windows in a specific session
tmux list-panes                     # list panes in the current window
```

## Config File (`~/.tmux.conf`)

Customize defaults by creating `~/.tmux.conf`:
```bash
# Change prefix from Ctrl+b to Ctrl+a (many people find this easier to reach)
unbind C-b
set -g prefix C-a
bind C-a send-prefix

# Enable mouse support (click to switch panes, drag to resize, scroll wheel works naturally)
set -g mouse on

# Use vi-style keybindings in copy mode (h/j/k/l navigation instead of arrow keys)
setw -g mode-keys vi

# Start window/pane numbering at 1 instead of 0 (more intuitive on a keyboard)
set -g base-index 1
setw -g pane-base-index 1

# Increase scrollback buffer size (default is only 2000 lines)
set -g history-limit 10000
```
After editing, reload without restarting tmux:
```
PREFIX :source-file ~/.tmux.conf
```
Or from a regular terminal: `tmux source-file ~/.tmux.conf`

## tmux vs. `nohup`/`disown` (When to Use Which)

For a single background process with no need to reattach and interact with
it, `nohup command &` or `command & disown` is lighter-weight. Reach for
tmux specifically when you need to **come back and actually interact** with
the session later — checking progress, typing more commands, monitoring
multiple panes — not just letting a process run unattended.

## Quick Reference Summary

| Action | Keys |
|---|---|
| Detach | `PREFIX d` |
| New window | `PREFIX c` |
| Switch window | `PREFIX n` / `PREFIX p` |
| Split vertical | `PREFIX %` |
| Split horizontal | `PREFIX "` |
| Switch pane | `PREFIX` + arrow key |
| Zoom pane | `PREFIX z` |
| Copy mode | `PREFIX [` |
| Paste | `PREFIX ]` |