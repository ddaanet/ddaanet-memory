---
name: feedback_design_supersede_no_museum
description: "when reversing a design decision, keep the load-bearing incident/rationale but cut deletion inventories, \"no longer does X\" framing, and change-bookkeeping; git diff already records what was cut"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 77fc3f96-023a-482a-be11-301d61bd7f4a
---

When writing a dated section that reverses an earlier design decision,
state what the design **is** and **why**, grounded in the load-bearing
fact (e.g. the incident that motivated the change). Cut the "museum
narrative": deletion inventories ("extract.py loses functions A, B,
C…"), "it no longer does what it used to" framing, and
inversion-bookkeeping ("this supersedes decision X and Y"). Terse
superseding *pointers* on the old sections are fine (a redirect stating
current truth); "retained as a record of what was cut" is not.

**Why:** the historical incident is what justifies the design and must
survive; the diff-narrative of what was removed is noise that a reader
mining for rationale has to wade through. The git diff already records
what was deleted.

**How to apply:** in a superseding DESIGN.md section, lead with the
motivating fact, then the positive design and its reasoning. Put the
"what changed" bookkeeping into the one-line superseding pointer, not
the prose. Refines [[feedback_design_doc_immutability]] (keep old
sections, add superseding note) and shares the spirit of
[[feedback_compact_prose]] (cut connective/re-summary wordage, keep
substance).
