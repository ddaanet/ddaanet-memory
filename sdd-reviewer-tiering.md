---
name: sdd-reviewer-tiering
description: Model-tiering rule for subagent-driven-development — reviewer one tier above implementer
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 0debf046-b60e-4aa0-ae82-36d3ef419595
---

When running `superpowers:subagent-driven-development`, pick the reviewer's model one
tier above the implementer's, not the same tier. E.g. sonnet implementer → opus
reviewer. Don't default both to the same model just because the skill's own
"least powerful model that can handle each role" guidance doesn't spell this out.

**Why:** my human partner flagged (2026-07-06, during the `display-test-system-plan` execution) that
running a big, judgment-heavy implementation task (Task 3: ~18 scenarios, dense visual
eyeballing of rendered grids) on sonnet felt optimistic given sonnet's context-handling
limits. The resolution: sonnet is fine for the implementer *if* the reviewer sits a tier
higher, because review is where subtle mistakes (a model convincing itself something
"looks right" when it doesn't) actually get caught — a same-tier reviewer is liable to
share the same blind spots as the generator. my human partner confirmed this explicitly as "a
generally valid tiering strategy" (reviewer.rank == generator.rank + 1), not just
acceptable for this one task.

**How to apply:** When dispatching per-task implementer + reviewer pairs in SDD (or
similar fork/review loops), default reviewer model to one rung above whatever model the
implementer used, especially for tasks involving dense pattern-matching/visual judgment
(rendered output, generated data, large diffs) rather than pure mechanical transcription.
Cheap/mechanical single-file fixes can still get a same-tier or even lighter review — the
rule matters most when the implementation itself required real judgment.
