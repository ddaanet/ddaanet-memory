---
name: plan-length-matches-work
description: "tracks the executable work, not the topic"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: ce981dd4-19be-4e5d-baa0-0c4f080d1af5
  modified: 2026-08-10T19:01:07.593Z
---

When the work is concrete ("edit N files + run M commands"), the plan doc should be that small. Padding with architecture diagrams, lessons-learned openers, multi-tier dogfood narratives, and self-review checklists feels productive but adds review burden without changing the work.

**Why:** exposition does not help an agent execute — it already has the context — and it defeats review, because nobody eyeballs 700 lines. A packaging plan reducing to 7 executable steps (edit two manifests, validate, push twice, dogfood twice) ran to 700 lines under a spec template inherited from the design-bearing plans before it.

**How to apply:** before writing a plan doc, list the actual executable steps. Under ~10 commands with no genuine design decisions, skip the spec-style template — a checklist of steps, open decisions and scope notes (≤30 lines) is plenty. Reserve full spec docs for plans carrying design decisions worth recording across sessions. The template fitting an earlier plan in the same series is not evidence it fits this one.

Related: [[large-docs-review]] (large docs defeat eyeball review).
