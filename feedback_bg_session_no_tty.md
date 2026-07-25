---
name: feedback-bg-session-no-tty
description: "background sessions can't launch interactive TUIs (revdiff: no /dev/tty); have the user run them and read the history/output instead of retrying the launcher"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 6afc4947-de16-4c6c-9503-de03977fa611
  modified: 2026-07-25T11:49:31.463Z
---

In a background Claude Code session (`CLAUDE_JOB_DIR` set), there is no
controlling terminal: revdiff failed with `could not open a new TTY:
open /dev/tty: no such device or address`, and the Zellij floating pane
the launcher spawns inherits the same problem (opens, revdiff exits
instantly, exit 1).

**Why:** interactive TUIs need a real terminal the user is viewing; a
detached/headless job can't provide one. Retrying the launcher just
repeats the failure.

**How to apply:** don't fight the launcher. Have the user run the TUI
in their own interactive terminal (e.g. `revdiff --untracked`), then
read their annotations from the history dir
(`~/.config/revdiff/history/<repo>/`) via revdiff's
`read-latest-history.sh`. No new history file = quit without
annotations = approved. Same class of limitation as
[[feedback_tmux_cc_popup]] (overlay not surfacing), but here the cause
is the missing tty, not the multiplexer mode.
