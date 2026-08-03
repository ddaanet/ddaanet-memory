---
name: run-the-gate-not-a-suite-subset
description: "verifying a change before declaring it done: run the repo's gate recipe, not the suites you believe are affected"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 6e0477fe-1f1f-42e9-bd70-94fc717a7601
  modified: 2026-08-03T15:14:53.336Z
---

When a repo has a gate recipe — `just precommit`, `just green` — run it to
verify a change. Do not assemble a hand-picked list of suites.

**Why:** picking the suites is a guess about blast radius, and removing
that guess is what the gate is for. The guess is worst exactly when a
change touches a shared library, which is when a subset looks most
obviously sufficient. The cost being avoided is smaller than it looks:
these gates cache on an input hash, so a re-run with nothing changed is
free, and the full price is paid only when something actually moved.

**How to apply:** run a single suite while iterating mid-change, and the
gate before saying the work is done. A slow figure recorded in the design
doc — gitlore's NFR10 puts `just precommit` at 530 s — describes work
worth optimizing, not permission to skip it. Related:
[[gate-cache-must-cover-every-check]], [[green-is-not-evidence]].
