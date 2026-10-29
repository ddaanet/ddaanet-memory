---
name: main-thread-delegates-diff-review
description: "in a dispatch loop the main thread does not read or review diffs itself: a subagent reviews each green diff; the main thread only orchestrates, runs the gate, commits and talks to my human partner"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 48dcf726-265b-452a-9406-f33306bc0ffe
  modified: 2026-10-29T19:44:58.228Z
---

In a red/green dispatch loop, the main thread does not read implementation
diffs or review them. A reviewer subagent does that and reports to a file;
the main thread orchestrates the agents, starts the precommit gate, commits,
and talks to my human partner.

**Why:** said on 2026-10-29, during the D70 branch in gitlore, when a plan's
Dispatch section had the main thread "review each diff": "don't review diffs,
delegate, you just orchestrate and talk to the human." Reading a diff fills
the main context, which has to stay free to hold the decisions and the
conversation across the whole loop.

**How to apply:** after a green phase, dispatch a Sonnet reviewer (one rung
above a Haiku implementer) with the contract and the reports' paths, and wait
for its verdict file. Act on the verdict: send fixes back to the implementer
or commit. A plan that names the main thread as diff reviewer is overridden by
this. Related: [[subagent-reviewer-tier]].
