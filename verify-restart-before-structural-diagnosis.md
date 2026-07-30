---
name: verify-restart-before-structural-diagnosis
description: "A same-session negative result (PATH entry missing, new agent/tool not visible) often just needs a harness restart, not a structural diagnosis"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 109169bb-771b-4bf9-9161-31c5e4dbb4ed
  modified: 2026-07-24T17:49:17.489Z
---

When something that should work doesn't resolve mid-session — a PATH
entry from a `.envrc` edit, a newly-created `.claude/agents/*.md` file,
any config the harness reads once at startup — check whether the
harness (Claude Code session, subagent registry, shell environment)
predates the change before concluding the mechanism is structurally
broken.

**Why:** during ghmem subproject 3 ([[ghmem-project]]), I diagnosed
bare `ghmem` failing to resolve in a freshly spawned subagent as proof
that `PATH_add` categorically cannot work for agent Bash calls — I
built a whole story about `~/.bashrc` gating direnv's hook on
interactive shells and the Bash tool always being non-interactive, and
shipped a workaround (hardcoded venv paths in agent prose, a widened
hook regex) before checking the simpler explanation. My human partner
corrected it: it just needed a Claude Code restart, exactly like a new
agent definition needing one — the session's Bash-tool environment was
fixed at a point before the `.envrc` edit, for every Bash call in that
session including subagent spawns, and no amount of respawning
sub-agents *within the same session* would ever see the edit. A
restart fixed both the PATH and the new agent's visibility in one step.

**How to apply:** before writing a root-cause diagnosis (and especially
before building a workaround around it) for "X doesn't resolve/appear
even in a fresh subagent", ask whether X depends on something read once
at process/session start (PATH baked at shell init, an agent registry
snapshotted at session start, cached config). A fresh *subagent* spawned
inside an existing session is not the same as a fresh *session* — it
can still inherit stale session-level state. If restart-vs-structural is
ambiguous, say so and ask, rather than shipping a workaround for a
misdiagnosed structural cause.
