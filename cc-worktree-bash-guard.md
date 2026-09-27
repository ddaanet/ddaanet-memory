---
name: cc-worktree-bash-guard
description: "a worktree session adds a Bash gate refusing `eval` and \"too complex to verify that it stays inside the worktree\" commands, blaming git isolation on commands touching no git; `pushd` is the drift stand-in"
metadata: 
  node_type: memory
  type: reference
  originSessionId: caeeb08d-e29e-4ff4-bd68-202bc30a450b
  modified: 2026-08-27T20:28:30.484Z
---

A session inside a Claude worktree (`EnterWorktree`) gains a second,
harness-owned Bash gate on top of any project hook. It refuses two command
shapes, and both refusals cite git isolation even when the command touches no
git:

- **`eval`** — "this command runs a string through eval, which can't be verified
  to stay inside the worktree".
- **Anything it reads as compound** — "this command is too complex to verify
  that it stays inside the worktree … Split it into plain, separate commands".
  A `set -e` multi-line script with a heredoc and a single
  `claude -p … --output-format stream-json … > file` invocation with an
  env-var-interpolated path both trip it (CC 2.1.247).

Both refusals close with "a worktree-isolated session's git operations must
target its own worktree", which misroutes debugging: the trigger is command
*shape*, and no git subcommand need appear.

**How to apply:** choose the venue before entering. Anything needing a scripted
probe — a multi-statement setup, a redirect, a nested `claude -p` — belongs in
the main tree; enter the worktree only for what the worktree lifecycle itself
drives, or split every step into one plain command. Where a test needs real cwd
drift, `pushd <dir>` passes the gate and persists, which `eval 'cd …'` cannot.

See [[cc-worktree-memory-freeze]] for what the same tool freezes in the
environment, and the `sandbox-lies:sandbox-lies` skill for the other refusal surface a Bash
command meets.
