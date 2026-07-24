---
name: Hook deny messages must not contain actionable phrases
description: "agent-facing deny messages must not contain actionable phrases; agents read \"if you need X, run Y\" as instructions. Scope: DENY channels only — DIRECTIVE additionalContext (agent meant to act) SHOULD be imperative; don't over-apply"
type: feedback
originSessionId: ab941ff0-5078-427e-b406-fba4e7538fc6
---
When `permissionDecisionReason` (or any agent-facing hook message)
denies a tool call, never include phrases like:

- "stop and report it to the user" → agent may try to invoke a
  reporting tool
- "if you really need X, run Y" → agent treats it as authorisation
  to run Y
- "revert via git rather than editing here" → agent runs git revert
- "remove the lock if stale" (the canonical example) → agent removes
  the lock immediately

**Why:** Agents lack the contextual filter that lets a human read
diagnostic prose as descriptive ("here's the situation") rather than
prescriptive ("here's what to do"). Any imperative phrasing in a
deny-reason gets executed as a literal instruction. This is distinct
from the broader "harness over agent" rule — even when work is
correctly in a hook, the *wording* of the agent-facing output still
needs special care.

**How to apply:**

- Phrase deny reasons as unconditional refusal + redirect to the
  legitimate path (e.g. "the release recipe owns this — invoke it
  instead of editing the file").
- Never offer escape hatches the agent can self-authorise.
- Use a separate `systemMessage` for human-facing detail; keep it
  curt (one line).
- If you find yourself writing "if you ... then ..." in an agent
  channel, rewrite as "do not ..." instead.

**Scope — this rule governs DENY channels, not DIRECTIVE channels.**
A `PreToolUse` deny is meant to *stop* the agent, so any actionable
phrase is harmful (the agent runs it). But `additionalContext` used as
a *directive* — where the whole point is that the agent should act
(present a line to the user, summarize+commit per a probe's output) —
is the opposite case: imperative wording is correct and required
there. See [[feedback-hook-agent-feedback]] (additionalContext exists
precisely to carry agent-actionable content). Do not over-apply the
no-actionable-phrases rule to directive output. Tell: is this channel
trying to halt the agent (deny → descriptive only) or drive it
(directive → imperative is fine)? A reviewer flagged handoff's
non-tmux `/rename` `additionalContext` as a violation; it was not —
that channel directs the agent to relay the line, and the probe's own
summarize→approve→commit directive is equally imperative by design.
