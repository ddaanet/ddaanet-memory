---
name: hook-output-channels
description: "`systemMessage` style is lowercase, em-dash, factual, no \"to fix this, run X\", and it renders ANSI so a warning line can lead with `\\033[0m`; an agent-facing DENY takes no actionable phrases (\"if you really need X, run Y\" reads as authorisation, \"remove the lock if stale\" gets obeyed) — but a DIRECTIVE `additionalContext` is imperative by design, so do not over-apply; channel mechanics and the permission pipeline are the `plugin-craft:hook-authoring` skill"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 618e80ab-c1a4-41c0-94c9-07e03fa4af68
  modified: 2026-08-27T11:47:42.000Z
---

Wording for the two audiences a hook writes to. The channel mechanics,
the `PreToolUse` permission pipeline, `updatedInput`/`updatedToolOutput` and
the stdin payload are the `plugin-craft:hook-authoring` skill.

### The user-facing channel: curt and informative

`systemMessage` and similar user-facing output: **lowercase, em-dash separator,
factual, no instructions or actionable phrases.** One short line — enough for the
user to know what happened, no more. My human partner named this style in the
2026-05-19 SessionStart-hook brainstorm ("style for user visible systemWarning
(and similar) is curt and informative"). No "to fix this, run X" — that is the
agent's job via `additionalContext`, not the user's. Matching examples:

- `handoff loaded — 3.2 KiB, saved 8m ago`
- `handoff: wiped prior handoff-task.md, handoff.md`
- `handoff extracted to /path/to/handoff.md`

`_wipe-emit.sh` is the in-repo reference implementation of this style — read it
rather than re-deriving the conventions.

**`systemMessage` renders ANSI escapes rather than showing them as literal
bytes.** So a hook that speaks only when something went wrong can lead its line
with a style reset (`$'\033[0m'`) to escape the dimming applied to ordinary hook
chatter, which is what handoff's drift report and context-size nudge do. Curt
and informative still governs the words; this is only about not being rendered
as noise.

### An agent-facing DENY: no actionable phrases

When `permissionDecisionReason` (or any agent-facing message) denies a tool call,
never include phrases like "stop and report it to the user" (agent tries to
invoke a reporting tool), "if you really need X, run Y" (agent treats it as
authorisation), "revert via git rather than editing here" (agent runs git
revert), or the canonical "remove the lock if stale" (agent removes the lock
immediately).

**Why:** agents lack the contextual filter that lets a human read diagnostic
prose as descriptive rather than prescriptive. Any imperative phrasing in a
deny-reason gets executed literally. Distinct from harness-over-agent: even when
the work is correctly in a hook, the *wording* of agent-facing output still needs
care.

**How to apply:** phrase deny reasons as unconditional refusal + redirect to the
legitimate path ("the release recipe owns this — invoke it instead of editing the
file"). Never offer escape hatches the agent can self-authorise. Put human-facing
detail in a separate curt `systemMessage`. If you write "if you … then …" in an
agent channel, rewrite as "do not …".

**Scope — this rule governs DENY channels, not DIRECTIVE channels.** A
`PreToolUse` deny is meant to *stop* the agent, so any actionable phrase is
harmful. But `additionalContext` used as a *directive* — where the whole point is
that the agent acts (present a line to the user, summarize+commit per a probe's
output) — is the opposite case: **imperative wording is correct and required
there.** Do not over-apply. Tell: is this channel trying to halt the agent (deny
→ descriptive only) or drive it (directive → imperative is fine)? A reviewer
flagged handoff's non-tmux `/rename` `additionalContext` as a violation; it was
not — that channel directs the agent to relay the line, and the probe's own
summarize→approve→commit directive is equally imperative by design.
