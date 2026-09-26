---
name: sandbox-effects
description: "zellij \"no active session\" from the Bash tool (its IPC socket is outside the command sandbox); every other sandbox effect — phantom dotfiles, lock errors, read-only paths, `$TMPDIR`, `excludedCommands`, escapes and their costs — belongs to the sandbox-lies plugin (its hooks annotate the symptom; its `sandbox-lies:sandbox-lies` skill carries the rest)"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 618e80ab-c1a4-41c0-94c9-07e03fa4af68
  modified: 2026-09-26T16:17:51.587Z
---

The Claude Code command sandbox distorts git, filesystem, transcript and IPC
operations. The sandbox-lies plugin owns that knowledge: its hooks annotate a
Bash result showing a known symptom, and its `sandbox-lies:sandbox-lies` skill
carries what has no symptom in the output — `excludedCommands` semantics, the
escapes and their costs, `$TMPDIR` scoping, cleanliness gates, merge recovery,
`ps`, `claude -p` and `--resume` under the sandbox. The zellij effect has no
detector; this file carries it.

## zellij IPC socket is outside the sandbox

In a zellij session (`ZELLIJ_SESSION_NAME=...`), calling `zellij run
--floating`, `zellij action new-pane`, etc. from the Bash tool returns "There
is no active session!" under the default sandbox, even with explicit `--session
<name>`. The session exists (`zellij list-sessions` shows "current") but
zellij's IPC unix socket (typically under `/tmp/zellij-<uid>/zellij-<session>/`)
is not reachable from the sandbox.

**Fix:** call Bash with `dangerouslyDisableSandbox: true`. The first symptom is
"no active session" from any zellij subcommand that needs to talk to the
running daemon (verified with `zellij --version` 0.44.3).

**Affects:** the `/revdiff:revdiff` skill launcher (`launch-revdiff.sh`) — it
picks the zellij branch when `ZELLIJ` is set and falls silently because the
launcher pipes `zellij run` stderr to `/dev/null`, so sandbox failure looks like
"exit 1 with no output"; and any future skill/script driving zellij from inside
CC. Once sandbox-disabled, the floating pane appears immediately.
