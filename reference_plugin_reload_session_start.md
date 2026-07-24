---
name: reference-plugin-reload-session-start
description: "skill bodies and hooks are snapshotted at session start; a mid-session plugin edit is NOT live until `/reload-plugins`"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 94e0a066-a634-4a25-a32d-b0f53b992c25
  modified: 2026-07-20T17:58:23.368Z
---

Editing a plugin's own files mid-session does **not** make the change live in
that session. Observed directly 2026-07-20: after rewriting
`skills/precompact/SKILL.md` and committing it, invoking
`/handoff:precompact` in the same session injected the **pre-rewrite** body,
even though the skill's reported base directory was the working tree
(`/Users/david/code/handoff/skills/precompact`) and the file on disk held the
new text.

So the loaded content is snapshotted at session start, not read at invocation.
By the same mechanism `hooks/hooks.json` is near-certainly session-start-loaded
too — **inferred, not directly observed**: newly-declared hook events (a fresh
`Stop` entry, a widened `SessionStart` matcher) should be assumed inactive
until a new session. Verify before relying on it either way.

**Why it matters:** when developing a plugin *in* the session that uses it, a
new magic file can be written with no hook registered to consume it. That is a
silent no-op, not an error — nothing fires, nothing complains, and the failure
looks like the feature simply not working.

**The fix is `/reload-plugins`** — no session restart needed. Run 2026-07-20 it
reported `18 plugins · 8 skills · 11 agents · 34 hooks`, and the next
invocation of the edited skill loaded the **new** body (the skill listing's
`description:` updated too). So the snapshot is refreshable on demand.

**How to apply:** when self-testing plugin changes mid-session, run
`/reload-plugins` first — the user can invoke it with `! /reload-plugins`, or
ask them to. Until then, fall back to the previously-shipped path and say so;
never report a mid-session plugin edit as "now active" without a reload.
Bundled *scripts* invoked by a bare-name `bin/` shim are the exception — PATH
is set at session start but points at the live directory, so script edits take
effect immediately ([[reference_skill_bundled_scripts]]).
