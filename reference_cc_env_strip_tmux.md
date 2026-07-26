---
name: reference-cc-env-strip-tmux
description: "TMUX/TMUX_PANE/TERM_PROGRAM/SSH_CONNECTION are dropped from the session env (rest inherits), so hook-driven send-keys finds no pane; `tmux display -p` guesses the wrong one — capture via a launcher shim"
metadata: 
  node_type: memory
  type: reference
  originSessionId: eafdf559-5d12-4139-8ed1-ba4a6095a9f6
  modified: 2026-07-25T12:11:08.670Z
---

> **Superseded 2026-07-25 (later same day):** retested after the user
> restarted and updated Claude Code to 2.1.220 — all five vars
> (`SSH_CONNECTION`, `TERM_PROGRAM`, `TERM_PROGRAM_VERSION`, `TMUX`,
> `TMUX_PANE`) were present in the Bash tool's `env`, unstripped. The
> strip below no longer reproduces on 2.1.220. Exact fix version unknown
> — the original measurement didn't pin a build below 2.1.x. Re-check
> `env` before relying on the launcher-shim workaround; it may no longer
> be needed.

Claude Code scrubbed terminal-provenance variables from the session
environment as of some build ≤2.1.x. Measured 2026-07-25 (tmux session) by diffing
`env` from the launching shell against `env` from both the Bash tool and
the `!` prompt shell — the two agreed exactly, and only five keys were
missing:

```
SSH_CONNECTION  TERM_PROGRAM  TERM_PROGRAM_VERSION  TMUX  TMUX_PANE
```

Everything else inherited, and ~44 keys were added (`CLAUDE_*`, proxy
vars, `SANDBOX_RUNTIME`, `TMPDIR`). So this is a denylist by name, not a
fresh environment — do not diagnose it as a detached or headless spawn.
Hooks see the same absence: they inherit from the same process, and
handoff's rename hook reported "not in tmux" from a genuine tmux session.

**Consequence.** Any hook-driven `tmux send-keys` automation has no pane
to target and must fall back to telling the user to paste.

**The tempting fix is wrong.** `tmux display -p '#{pane_id}'` answers
from the default socket with the server's last-active pane, which is a
guess: measured together, the real pane was `%13` while `display -p`
returned `%11` — a different session's pane, which the automation would
have typed into. `tmux list-panes` with no target misleads the same way,
listing one pane and looking authoritative. Treat a tmux command run
without `$TMUX` as answering about *some* session, never this one; same
default-socket fallback as [[reference_tmux_test_isolation]].

**How to apply.** Capture the pane in a launcher shim that runs before
`claude`, and pass it as an env var under a name outside the strip list:

```sh
exec env HANDOFF_TMUX="$TMUX" HANDOFF_TMUX_PANE="$TMUX_PANE" claude "$@"
```

Carry `$TMUX` as well as the pane — its first comma-field is the socket
path (`/tmp/tmux-1000/default,772,0`), needed as `tmux -S` on a
non-default socket. Prefer the env var over a file: the shim runs before
the session id exists, so it cannot name a session-scoped file, and a
fixed path is clobbered by a second session launched from a second pane
— reinstating the wrong-pane bug. Keep the paste fallback as the loud
path, since the strip is a denylist today and an allowlist or prefix rule
would break the shim silently.
