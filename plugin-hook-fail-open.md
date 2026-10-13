---
name: plugin-hook-fail-open
description: "a plugin hook registered session-wide (`hooks/hooks.json`, every Bash/Read/Write/Edit) must fail open silently when its own deps (`jq`, GNU `realpath`) are missing — report them at SessionStart, never `trap … exit 2` / fail closed"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 36adc479-acd7-4d21-9a41-9e5db0a3cebf
  modified: 2026-10-13T18:01:54.251Z
---

A hook a plugin registers for the whole session — every matching tool call
of every session that enables the plugin, not just the plugin's own agents —
fails **open, silently**, when it cannot run: a missing `jq`, a `realpath`
without GNU's `-m`, a payload it cannot parse. Missing dependencies are
reported once, at `SessionStart`, on a channel both the user and the agent
see (`systemMessage` plus `additionalContext`).

**Why:** my human partner rejected a fail-closed design (an `ERR` trap
turning any hook failure into exit 2) for the ghmem containment hooks:
on a machine without `jq` it refused Bash, Read, Write and Edit in every
session with the plugin enabled. Bricking the session is not an acceptable
outcome, even for a containment hook guarding agents that read untrusted
text. Visibility comes from the SessionStart report instead of from
blocking.

**How to apply:** in a session-wide hook script, any failure before the
hook has decided ends in a silent `exit 0`, and nothing in its pass-through
path writes to stderr (a non-2 exit with stderr shows as a hook error on
every call). Ship a `SessionStart` hook in the same plugin that checks
the dependencies and names any that are missing, with what stops working.
The agent-side containment self-check (a must-block tripwire the agent runs
first) is what stops an uncontained agent; see
[[plugin-agent-hooks-ignored]].
