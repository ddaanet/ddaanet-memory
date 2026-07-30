---
name: turn-call-terminology
description: "\"turn\" = a *user* turn, not an API round-trip and not a tool call; \"call\" = an API or tool call"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 8cd21851-d241-49bf-afaf-b4baf0c4f311
  modified: 2026-07-25T19:07:22.775Z
---

A **turn** is a *user* turn: one user message and everything that follows it
until the next user message. It is not an API round-trip and not a tool call. A
**call** is either an API call or a tool call; which of the two is usually left
unstated, because in most contexts the distinction does not change anything.

**Why:** established while reviewing handoff's with-commit memory directive,
whose text claims "this turn's commit carries the memory". Whether that is true
depends entirely on what a turn is — and under the user-turn reading, a
`/handoff` followed by a `/commit` is two turns, so the claim is false for the
very flow the skill names as that mode's routine case. Terminology that shifts
under a reviewer's feet makes a finding unarguable in either direction.

**How to apply:** reserve "turn" for user turns, in prose and in skill/directive
text alike; when the boundary matters, say which user message starts it. Use
"call" for API and tool calls and do not disambiguate unless the difference is
load-bearing. Related: [[cc-tui-tmux-driving]], which depends on this
vocabulary throughout.
