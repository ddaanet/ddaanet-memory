---
name: feedback-subagents-over-applied
description: "one-subagent-per-task is mis-granular; the axis is batching (one agent for several small/mechanical/one-loop tasks to spare main-session context), not agents-vs-inline; a shared mutable file only rules out *parallel* agents (mechanical does not)"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 234f3903-010b-4ab9-9675-6b2c7977657e
---

When an implementation plan says "execute via subagent-driven-development"
with one subagent per task, critique the *granularity* before dispatching.
"N tasks" ≠ "N subagents" — but that does not mean "N tasks → inline."

**Why:** The real cost a subagent offsets is **context churn in the main
session**, and it pays that off only when the delegated chunk is big enough.
The common error is granularity, not the choice to use agents:

- A repeat mechanical edit does **not** disqualify an agent. Each individual
  edit may not warrant its *own* agent, yet grouping all of them into a
  *single* agent can make sense — it keeps the churn out of the main session.
- One TDD red/green loop rarely pays for its own agent. But a single TDD
  agent can carry *several* one-loop tasks in sequence.
- A shared mutable file (e.g. every task appends to one `hook-test.bats`) only
  rules out *parallel* agents — one sequential agent handling the whole batch
  is fine.

**How to apply:** Keep the plan's commit boundaries (good hygiene), but choose
agent boundaries by batching, not by task. Ask "which contiguous run of small
tasks can one agent own end-to-end to spare main-session context?" rather than
"does this task need an agent?" Going fully inline (as on the per-worktree
handoff root work, [[project-worktree-handoff-root]]: 6 proposed agents → 0
used) is one valid answer when the total surface is small; a single agent
batching tasks 2–6 would have been another. The TDD red/green discipline holds
either way — [[feedback-tdd-red-green-separation]] is about not *bundling* red
into a green agent, not about agent count.
