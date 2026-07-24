---
name: living design doc structure
description: Design docs follow a specific pattern with six sections
type: feedback
originSessionId: 44430e43-5ab7-4730-8696-8993292db475
modified: 2026-07-24T17:52:54.972Z
---
Design documents are living docs, not one-off specs. Structure: functional requirements, non-functional requirements, architecture, design decisions (with rationale), rejected alternatives, changelog.

**Why:** Captures the "why" alongside the "what" so future sessions don't re-litigate settled decisions. The rejected alternatives section is especially important — it prevents revisiting approaches that were already ruled out with good reason.

**How to apply:** When updating `docs/design.md`, add to the changelog and update the relevant section. Don't flatten or summarize decisions — preserve the rationale. Plans go in `docs/plans/`, specs/design docs in `docs/design.md` (single living doc for this project).

Related: [[design-doc-immutability]] (a past decision's rationale can still be re-examined and inverted), [[feedback_design_supersede_no_museum]] (how to write the dated section when a decision is reversed).
