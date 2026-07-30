---
name: stale-artifact-is-not-agent-death
description: "A subagent's unwritten report proves nothing about whether it is alive; re-dispatching on that inference put two implementers in one tree and corrupted both their verification runs"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 8bc34870-8b8c-49fa-85a0-4ad6d99085e8
  modified: 2026-07-21T21:06:22.736Z
---

On 2026-07-21, mid-SDD, I checked whether a Sonnet implementer had
finished by looking at its report file's mtime. The file was still the
stale version, so I concluded the agent had died in the machine's
recent reboot and dispatched a replacement into the same working tree.

It had not died. It was still working, and wrote a complete, good report
minutes later. For a while **two implementers were editing
`heat_query.py` concurrently** — and both were running the brief's
mutation-testing step, deleting a call from the file and re-running
pytest. Each therefore ran its test suite while the other had the file
mutated, so **both agents' "just green is clean" claims were worthless**,
and one of them wrote a puzzled report section theorising about
out-of-order harness notifications when it had simply observed its peer's
edit.

Nothing was lost, but only by luck: the final on-disk state happened to
be coherent.

**Why:** an output artifact is written at the *end* of a task, so its
absence is equally consistent with "dead" and "still working" — it
carries almost no signal, and the two cases call for opposite actions.
The failure is silent and mutually destructive: concurrent agents in one
tree do not detect each other, they just quietly invalidate each other's
evidence.

**How to apply:** never infer a subagent is dead from a missing or stale
output file. Prefer waiting — the harness re-invokes on completion, so
polling is usually wasted anyway. Also reject weak liveness proxies: I
next tried load average, and my human partner pointed out it was their
other sessions, not my agent. Before re-dispatching into a tree that
already has an agent in it, get positive evidence the first one is gone,
and if you cannot, ask rather than guess. When a replacement does land on
inherited work, re-verify the gate yourself in a quiet tree instead of
trusting any agent's green claim — see
[[machine-resource-limits]] for why agents here go silent in the first
place, and [[mutate-code-to-test-the-tests]] for why a mutation run in a
contended tree is meaningless.
