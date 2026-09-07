---
name: hook-output-channels
description: "`systemMessage` renders ANSI, so a hook that speaks only on trouble leads with `\\033[0m` to escape the dimming applied to ordinary hook chatter, and `_wipe-emit.sh` is the in-repo reference implementation of the curt style; channel mechanics, the `PreToolUse` permission pipeline and `updatedInput`/`updatedToolOutput` are the `plugin-craft:hook-authoring` skill, and the wording rules — curt and factual for the user, no actionable phrases in an agent-facing deny, imperative for a directive — are the `craft:directive-writing` skill"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 618e80ab-c1a4-41c0-94c9-07e03fa4af68
  modified: 2026-09-07T12:51:02.056Z
---

Where the two halves of hook output now live, and the one rendering mechanism
that belongs with neither.

The channel mechanics, the `PreToolUse` permission pipeline,
`updatedInput`/`updatedToolOutput` and the stdin payload are the
`plugin-craft:hook-authoring` skill.

The wording rules are the `craft:directive-writing` skill, which states them at
the moment they apply: curt and informative for a user-facing `systemMessage`,
no actionable phrases in an agent-facing deny, and the scope limiter that
exempts directive channels — an `additionalContext` meant to *drive* the agent
is imperative by design.

`_wipe-emit.sh` is the in-repo reference implementation of the curt user-facing
style — read it rather than re-deriving the conventions.

**`systemMessage` renders ANSI escapes rather than showing them as literal
bytes.** A hook that speaks only when something went wrong can lead its line
with a style reset (`$'\033[0m'`) to escape the dimming applied to ordinary
hook chatter — handoff's drift report and context-size nudge both do. Curt and
informative still governs the words; this is only about not being rendered as
noise.
