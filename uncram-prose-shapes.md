---
name: uncram-prose-shapes
description: "crammed prose has three tells: multi-rule blocks, semicolon chains, colon-joined fragments; keep load-bearing em-dashes"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 17958a2e-8286-4cc9-b9cd-32f0f0050c75
  modified: 2026-07-21T10:33:46.654Z
---

Prose written under a line cap crams in three recurring shapes: a
docstring stating four or five separate rules in one unbroken block;
contract documentation run together as a semicolon chain (`Keys: id;
itemId (**injected**: ...); author; ...`); and rationale telegraphed
down to clause fragments joined by colons. An "uncram pass" unpacks
these — one idea per paragraph, one annotated key per line — adding and
dropping nothing.

**Why:** Density under pressure is not concision. Every rule, exception
and reason survives the pass; only the packing changes. My human
partner asks for this as a distinct pass ("do the uncram pass on X"),
separate from any behaviour change.

**How to apply:** Split genuine pile-ups only. A single em-dash or an
explanatory colon is load-bearing — `Not its own table — nested JSON
inside ...` says more, in less, than two flat sentences, so flattening
it trades terse for limp and is a regression. Context paragraphs go
*before* a test docstring's `When:`/`Then:` block, never after. If the
pass pushes a file past its line cap, split the module along an
existing seam rather than shaving the prose back to fit — the cap protects
reviewability, and prose is the first thing a later reader needs. Do the
judgement yourself: an underspecified style brief affords only
find-and-replace, and subagents flatten the voice.
