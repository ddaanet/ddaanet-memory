---
name: subagent-skips-at-import-expansion
description: "an Agent-tool subagent's claudeMd omits CLAUDE.md's `@path` imports a real session resolves; verify with `claude -p`, not a subagent"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 088576b6-2351-4a78-b826-5679e810c7ce
  modified: 2026-08-11T13:41:24.305Z
---

An Agent-tool subagent's `claudeMd` context block includes the literal contents
of `CLAUDE.md` files, but does not expand `@path/to/file` import lines the way
a genuinely fresh top-level session does. A subagent asked to check whether an
`@` import resolved will report the imported file absent even when the import
is correctly written and does resolve for a real session.

Confirmed 2026-08-11: after adding `@memory/ddaanet/shared-claude.md` as the
final line of a project `CLAUDE.md`, a subagent dispatched via the `Agent`
tool reported the shared file's contents missing from its `claudeMd` block.
Running `claude -p "<check the same thing>"` from the same directory —
initializing a genuine new session — showed the import resolved correctly.

**How to apply:** to verify a CLAUDE.md `@` import actually resolves, use a
real fresh session (`claude -p '...'` from the target directory, or ask the
user to start one) rather than the `Agent` tool. A subagent's negative result
on this specific question is not evidence the import is broken.
