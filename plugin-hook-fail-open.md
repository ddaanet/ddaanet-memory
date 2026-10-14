---
name: plugin-hook-fail-open
description: "session-wide `hooks/hooks.json` containment hook: silent exit 0 until `agent_type` names its agent, exit 2 after; SessionStart check runs the hook, not `command -v`"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 36adc479-acd7-4d21-9a41-9e5db0a3cebf
  modified: 2026-10-14T12:48:00.805Z
---

A hook a plugin registers for the whole session runs on every matching tool
call of every session that enables the plugin, not just the plugin's own
agents. When it guards those agents, it **fails open, silently, until the
caller is identified, and fails closed after**:

- **Silent exit 0, nothing on stderr:** the payload does not parse;
  `agent_type` is absent, not a string, or not one of the hook's agents; the
  hook command itself is missing (the wrapper exits 0).
- **Exit 2 with a reason on stderr:** the hook has read `agent_type` as one
  of its own agents and then fails — a missing or mistyped `tool_input`,
  `command` or `file_path`, no project directory, a path that does not
  resolve, any exception. Only those agents stop.

**Why:** my human partner rejected failing closed everywhere (an `ERR` trap
turning any hook failure into exit 2): on a machine missing a dependency it
refused Bash, Read, Write and Edit in every session with the plugin enabled,
including sessions that never dispatch its agents. Narrowing the block to
identified agents does not bring that back.

**Precondition:** nothing after identification may depend on an external
tool. ghmem meets it with a stdlib-only Python console script
(`ghmem-hook`; the `hooks/scripts/*.sh` are one-line wrappers). The same rule
in bash would have blocked every file-tool call of both agents on macOS,
whose `realpath` has no `-m`. The one dependency left is the hook command
being on PATH and able to start.

**How to apply:**

- Ship a `SessionStart` hook that **runs** the hook command once, with a
  payload naming no agent, and expects exit 0. `command -v` is not enough: a
  command on PATH with a dead interpreter or a failing import exits
  126/127/1 before it can decide, and Claude Code reads a non-2 exit as "not
  a block", so every call passes. On failure, name it on `systemMessage` for
  the user and on `additionalContext` for the agent, telling it not to
  dispatch the contained agents.
- Fire that report on startup, resume, clear and compact alike: after
  `/clear` or `/compact` the agent's copy of the warning is gone.
- Have each contained agent open with a two-call tripwire: a Bash `echo` and
  a Read of `.gitignore`, both of which must be blocked. The Bash call alone
  cannot show that a file-tool hook did not load. See
  [[plugin-agent-hooks-ignored]].
