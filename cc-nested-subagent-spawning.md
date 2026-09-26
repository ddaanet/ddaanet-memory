---
name: cc-nested-subagent-spawning
description: "a subagent can call `Agent` (`CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH`, three layers verified) but the delegating layer must be `general-purpose`: `edify:*` agents enumerate tools without `Agent`, `claude` narrates background jobs; one opus delegate holds about one `/orchestrate` phase, its brief caps context at ~200k with a committed Resume section, phase-boundary resume blocks are the handoff unit; a skill's approval gate returns as the delegate's final message, answered with `SendMessage`"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 78c01ada-e3fc-450c-b044-3f914c0034dc
  modified: 2026-09-26T11:20:21.567Z
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
- **The brief sets a hard context stop at ~200k tokens** (my human partner's
  figure). Past it the delegate finishes only the dispatch or commit in
  flight, writes and commits a Resume section (last commit, exact slice step,
  decisions given, carry list) and stops. A `/build` Phase 2 delegate briefed
  without a cap ran to ~330k before anyone noticed; a delegate does not stop
  on its own, and a teammate's idle notices carry no token count.
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
