---
name: sdd-reviewer-tiering
description: "Sonnet is the reviewer/`claude-code-guide` floor (Haiku hallucinates); Fable designs, Opus drafts, Sonnet executes"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 0debf046-b60e-4aa0-ae82-36d3ef419595
  modified: 2026-07-31T20:09:59.286Z
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

**Sonnet is the floor for judgment roles, whatever the tiering says.** Haiku is
too weak to be a reliable reviewer: it misses subtle issues, and on
`claude-code-guide` queries it also omits useful information and occasionally
hallucinates. Dispatch every reviewer and every guide/research query at
`model: sonnet` or higher — never let one default to a smaller model. Haiku
remains fine for an implementer on mechanical, isolated tasks. My human partner
corrected both of these explicitly mid-session.

**The default shape of a multi-stage task**: Fable designs, Opus drafts, Sonnet
executes. Keep Fable turns lean and code-free — design decisions, verdicts,
fixes, prose outlines — because cheaper agents produce code fine from a prose
outline and Fable tokens are expensive. Plan drafting goes to an Opus subagent
with Fable reviewing the draft; execution goes to Sonnet in a later session.
Delegate file-producing work to subagents with self-contained prompts and
review their output against the spec instead of redoing it.
