---
name: tmux-cc-popup
description: tmux display-popup does not render in iTerm2 -CC (control) mode; use tmux new-window for interactive TUI overlays from Claude
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 7bed82ba-de53-4fd5-9e88-49823dea7260
---

When launching interactive TUI tools (revdiff, lazygit, etc.) from
Claude inside a tmux session, do not rely on `tmux display-popup`.
The user's tmux is typically attached via `tmux -CC new-session -A`
(iTerm2 control mode), where popup escape sequences are not
translated and the popup is created but invisible to any client.

**Why:** Symptom observed 2026-05-17 — revdiff launcher detected
`$TMUX` and used `display-popup`; user reported "where can I find
the review overlay?" because in -CC mode the popup goes nowhere.

**How to apply:** When the launcher you're using offers a tmux
backend, prefer `tmux new-window -d -n <name> -t <session>:` instead
of `display-popup`. In -CC mode the new window renders as a normal
iTerm2 tab; in classic mode it's reachable via `Ctrl-b w/n`.
Confirm CC mode with `$TERM_PROGRAM=tmux` while `$TMUX` is set; the
underlying terminal (e.g. iTerm.app) is hidden from env vars in
this state.

Related: [[revdiff-plugin]] for the revdiff-specific launcher path.
