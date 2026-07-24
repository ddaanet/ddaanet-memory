---
name: systemmessage-style
description: "User-visible hook output (systemMessage and similar) style is \"curt and informative\" — lowercase, em-dash, factual, no instructions"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 45acd379-a2c1-4cfc-8f7b-e506ad744d38
---

User-visible hook output (`systemMessage` and similar user-facing
channels) should be **curt and informative**: lowercase, em-dash
separator, factual, no instructions or actionable phrases. One short
line — enough for the user to know what happened, no more.

**Why:** the user explicitly named this style in the 2026-05-19
brainstorm for the SessionStart-hook design ("Remember style for
user visible systemWarning (and similar) is curt and informative.").
The existing `_wipe-emit.sh` is the in-repo reference:
`"handoff: wiped prior handoff-task.md, handoff.md"`.

**How to apply:** when designing a hook that emits `systemMessage`,
write the line in lowercase, factual voice, em-dash for separation,
no "to fix this, run X" — that's the agent's job via
`additionalContext`, not the user's. Examples that match:
- `handoff loaded — 3.2 KiB, saved 8m ago`
- `handoff: wiped prior handoff-task.md, handoff.md`
- `handoff extracted to /path/to/handoff.md`

Related: [[hook-deny-wording]] (the same principle for the
agent-facing `permissionDecisionReason`), [[hook-agent-feedback]]
(channel separation, user vs agent).
