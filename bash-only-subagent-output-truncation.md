---
name: bash-only-subagent-output-truncation
description: "a Bash-only contained subagent can't tell whether its own tool output was truncated on a large single command, and its self-report of how much it saw is not evidence of the real cutoff; fix is a file the command also writes plus granting `Read`, not a bigger Bash cap"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 87e146ca-b89c-467e-99ab-fa57fdc0a6a9
  modified: 2026-10-11T11:29:38.353Z
---

A subagent restricted to Bash only, with no other tool, cannot tell from
inside itself whether a single command's stdout was cut short by the
Bash tool's own output cap. Its self-report of how much it saw ("roughly
2KB", "the opening exchange") is a guess made from inside the very
output that was cut, not a measured cutoff — two independently
dispatched agents hitting the same real large output (one ~34KB, one
~42.5KB) each gave a different, imprecise estimate.

**Why:** ruling out the contained system's own internal size budget
first (confirming, independently, that the items were well under its
configured limits) isolates the cause as the Bash tool's own output
capture — a layer underneath the contained system, which the agent has
no visibility into and cannot reason about from a single call.

**How to apply:** don't raise the Bash output cap or try to measure its
exact value — design around it. Have the command whose output might run
long also write its full output to a file, name the file in a line the
agent can parse, and grant the subagent `Read` in addition to its gated
Bash tool. `Read` is non-mutating, so it doesn't reopen whatever a
Bash-allowlist hook exists to close; it only adds visibility. Tell the
agent to Read the file only when the printed output looks cut short
(ends mid-sentence, short of an expected count), not on every call.
