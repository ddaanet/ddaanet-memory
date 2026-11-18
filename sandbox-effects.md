---
name: sandbox-effects
description: "`There is no active session!` from `zellij run` under the Bash tool, or `/revdiff:revdiff` / `launch-revdiff.sh` exiting 1 with no output inside zellij (`ZELLIJ` set): the sandbox hides zellij's socket"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 618e80ab-c1a4-41c0-94c9-07e03fa4af68
  modified: 2026-11-17T22:58:22.000Z
---

The `/revdiff:revdiff` launcher (`launch-revdiff.sh`) takes its zellij branch
when `ZELLIJ` is set and pipes `zellij run`'s stderr to `/dev/null`, so under the
command sandbox it exits 1 with no output instead of showing zellij's "There is
no active session!". The cause is zellij's IPC socket sitting outside the
sandbox, and the fix is a Bash call with `dangerouslyDisableSandbox: true`
(verified with `zellij --version` 0.44.3); the mechanism is in the
`sandbox-lies:sandbox-lies` skill's `references/symptoms.md`. The same holds for
any skill or script that drives zellij from inside Claude Code.
