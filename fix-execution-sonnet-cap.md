---
name: fix-execution-sonnet-cap
description: "Model tier for dispatched agents in a review/fix cascade — reviewers inherit Opus, executing a well-specified fix is capped at Sonnet"
metadata:
  node_type: memory
  type: feedback
  originSessionId: a3b9c059-11d4-4698-9989-5d4321de1d38
  modified: 2026-11-27T10:46:24.966Z
---

An agent that executes a well-specified fix (the fix is named in the brief:
file, change, what must not change) runs on Sonnet at most, never Opus.
Review agents — the /simplify angles, quality reviewers — are not capped
down: leaving `model` unset (inherits Opus) is the default there, and
overriding them to Sonnet produced passes that sampled a handful of files out
of ~50 and missed most of the diff.

**Why:** my human partner, 2026-11-27, during a release /simplify pass: after
the reviewers had been sent on Sonnet, they pointed out that Opus is the
default for a reason; then, after the fix agents went out on Opus, "souviens toi:
plafond sonnet pour execution de fix bien specifiés". Judgment is spent on
finding; execution of a spelled-out fix does not need it.

**How to apply:** at dispatch, decide by the task's openness, not by the
previous dispatch's tier: reviewer → leave model unset; fix with the change
spelled out → `model: "sonnet"` (Haiku when fully mechanical); a fix whose
approach is still open → escalate per edify:dispatch's refactoring row.
