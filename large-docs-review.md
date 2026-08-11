---
name: large-docs-review
description: "flag high-risk sections explicitly rather than asking for a full eyeball pass"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: ce981dd4-19be-4e5d-baa0-0c4f080d1af5
  modified: 2026-08-11T20:02:14.299Z
---

When presenting a large planning or design document for user review, do NOT just commit it and ask "look this over." My human partner's reaction: "documents are too large for eyeballs" — large docs hide errors in volume, and reviewers (human or LLM) skim instead of inspect.

**Why:** Plan 04 (gitlore `plans/2026-05-22-04-marketplace-install.md`) was ~680 lines. My human partner's first revdiff pass surfaced exactly one annotation (a redundant test layer). They later said they "should have looked for changes and did not" — meaning the volume defeated the review.

**How to apply:**
- Before asking my human partner to review a long doc, do a fresh-eyes pass yourself and flag specific high-risk sections (concrete shell snippets, cross-repo paths, file lists, anything that requires checking against the world rather than reading prose).
- Present those sections explicitly: "Lines X–Y do Z — worth verifying" beats "review the whole plan."
- For doc edits, prefer `revdiff HEAD~1` (revision-only diff) over `HEAD~N` (cumulative) when my human partner just needs to verify a targeted change.
- Consider splitting structurally: spec → its own commit; implementation plan → its own commit. Smaller diffs review better than one large drop.
- The brainstorming skill's "spec self-review" step exists for exactly this reason — actually do it with rigor, don't just tick the checkboxes.

