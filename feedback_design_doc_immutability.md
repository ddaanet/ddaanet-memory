---
name: design-doc-immutability
description: "A design doc captures thinking at write-time, not the current best answer — when rationale looks thin on re-reading, re-examine and be willing to invert decisions"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 45acd379-a2c1-4cfc-8f7b-e506ad744d38
---

When a design doc's rationale for a past decision looks thin on
re-reading, **re-examine it instead of treating it as settled**. The
doc captured the thinking at the time, with the information available
then; new context (or fresh eyes) can invert old decisions.

**Why:** in the 2026-05-19 handoff brainstorm, the user pushed back
on the "no SessionStart hook, use `@`-ref instead" decision recorded
in `DESIGN.md:288-304`. On re-examination the original arguments
("no hook overhead", "no race conditions") were thin — both negligible
in practice — and the cost of the chosen path (a whole user-facing
setup skill, an entire silent-failure mode) was structural. The
inversion produced a cleaner architecture.

**How to apply:**
- When citing a past design decision, check whether the *reasons* it
  gave still hold. Don't just relay the conclusion.
- If the user challenges a documented decision, default to honest
  re-examination over defending the doc.
- Be willing to write a new spec that inverts an old one. The spec
  history (in git) is the audit trail; the latest doc is the current
  answer.

**Scope after a design inversion:** when reworking a doc to reflect an
inverted decision, distinguish the two things that look similar but
demand different treatment:

- **Rationale paragraphs** (why option A was chosen, what trade-offs
  were weighed) — preserve. These are the audit trail of write-time
  thinking and have ongoing value as design history.
- **Mechanism descriptions** (how the system behaves *now*, what
  happens in edge case X) — update. Leaving these stale doesn't
  preserve thinking, it actively misleads readers about current
  behavior.

In the 2026-05-19 v0.3.0 release, two reviewers split on this for
`DESIGN.md`'s `## Activation` chosen-pattern bullet (still described
the `@`-ref load chain) and `### Missing-extract edge case` (still
described `@`-ref failure). The spec reviewer invoked
design-doc-immutability to leave them; the code-quality reviewer
argued they were current-behavior descriptions, not rationale. The
code-quality framing was correct — the fix went in as commit
`7c0471b`.

Related: [[harness-over-agent]] (another design principle whose
*application* can shift even though the principle holds).
