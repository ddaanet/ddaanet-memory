---
name: plugin-hook-fail-open
description: "session-wide `hooks/hooks.json` containment hook: silent exit 0 until `agent_type` names its agent, exit 2 after — the shell wrapper identifies the caller itself, so a killed interpreter (137) or one exiting 1 blocks; SessionStart check runs the hook, not `command -v`"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 36adc479-acd7-4d21-9a41-9e5db0a3cebf
  modified: 2026-10-15T15:38:15.230Z
---

A hook a plugin registers for the whole session runs on every matching tool
call of every session that enables the plugin, not just the plugin's own
agents. When it guards those agents, it **fails open, silently, until the
caller is identified, and fails closed after**:

- **Silent exit 0, nothing on stderr:** the payload does not parse;
  `agent_type` is absent, not a string, or not one of the hook's agents —
  whether the module says so or the wrapper does because the module could not
  run. A signal death is the one visible case: exit 1, one line on stderr,
  which Claude Code shows as a hook error and does not act on.
- **Exit 2 with a reason on stderr:** the caller is one of the hook's agents
  and the hook cannot decide — a missing or mistyped `tool_input`, no project
  directory, any exception, and equally an interpreter that is missing, killed
  (OOM), exits anything but 0 or 2, or a module missing or unreadable. Only
  those agents stop.

**Why:** my human partner rejected failing closed everywhere (an `ERR` trap
turning any hook failure into exit 2): on a machine missing a dependency it
refused Bash, Read, Write and Edit in every session with the plugin enabled,
including sessions that never dispatch its agents. Narrowing the block to
identified agents does not bring that back. Identifying only inside the module
is not enough: a wrapper that passes through every status but 0 and 2 lets a
python3 killed mid-check wave an agent's call through.

**Precondition:** nothing after identification may depend on an external
tool. ghmem runs a stdlib-only module with `python3 -I` from the plugin's own
tree (`hooks/scripts/hook.sh <name>`), and the wrapper identifies the caller
with a bash regex on the raw payload, builtins only. The regex has no false
positive because every `"` inside a JSON string is escaped, so only a
structural `"agent_type":` key matches; it relies on Claude Code serializing
with `JSON.stringify`.

**How to apply:**

- Ship a `SessionStart` hook that **runs** the hook's checks, not
  `command -v`: per hook one call from an identified agent that must be
  allowed (exit 0), plus one identified call that must be blocked (exit 2). A
  payload naming no agent returns before any check runs, so it passes a broken
  interpreter. On failure, name it on `systemMessage` for the user and on
  `additionalContext` for the agent, telling it not to dispatch the contained
  agents.
- Fire that report on startup, resume, clear and compact alike: after
  `/clear` or `/compact` the agent's copy of the warning is gone.
- Have each contained agent open with a two-call tripwire: a Bash `echo` and
  a Read of `.gitignore`, both of which must be blocked. The Bash call alone
  cannot show that a file-tool hook did not load. See
  [[plugin-agent-hooks-ignored]].
