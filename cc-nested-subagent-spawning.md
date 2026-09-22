---
name: cc-nested-subagent-spawning
description: "subagents can spawn subagents (`CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH`), but the delegating layer must be `general-purpose` — `edify:*` agents lack `Agent`; one opus delegate holds one `/orchestrate` phase; skill gates return as its final message, answered by `SendMessage`"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 78c01ada-e3fc-450c-b044-3f914c0034dc
  modified: 2026-09-22T13:06:02.721Z
---

Verified against the Claude Code docs and a live probe on CC 2.1.278: an
`Agent`-tool subagent can itself call `Agent`, to a depth set by
`CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH` (three layers below the main
conversation worked), and background subagents keep the `Agent` tool.

Constraints on the delegating layer:

- **Its type must be `general-purpose`.** The `edify:*` agent definitions
  enumerate their tools and `Agent` is not among them, so a delegate of that
  type cannot dispatch. The `claude` type carries a background-job narration
  protocol that gets in the way of a skill running its own dispatch loop.
- **One opus delegate carries about one heavy `/orchestrate` phase** before
  its context fills. Hand a skill's work to a delegate in phase-sized units,
  and make the skill's phase-boundary resume block the handoff artefact — the
  next delegate starts from it, not from the previous one's transcript.
- **A skill's approval gate comes back as the delegate's final message.** The
  delegate stops there; the answer goes back with `SendMessage` to the same
  agent, which then continues with its context intact. A new `Agent` call
  would restart the skill from nothing.
- The delegate's Bash runs in the main session's sandbox, so a dispatch that
  has to fetch names its hosts in `allowed_domains` up front.
- A delegate that runs `git stash` to compare two states takes the parent's
  staged files with it; the dispatch prompt rules out `git stash`, `git reset`
  and blanket `git add`, and names any pre-staged files.

**Why:** running `/orchestrate` inside a delegate rather than the main
session keeps the main context free for review and the audit; the run this
was verified on went main session → `general-purpose` delegate invoking the
skill → per-item agents.
