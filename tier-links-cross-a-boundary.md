---
name: tier-links-cross-a-boundary
description: "routing a fact up carries its `[[links]]` verbatim, and ones aimed at project-local siblings go permanently dead, unlike a write-it-later dangle"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: c1666e33-a938-42c3-a372-e89af689e7f6
  modified: 2026-07-30T17:15:07.639Z
---

When a memory is routed from a project's own store up into a shared tier, its
`[[links]]` travel with it verbatim. Any that pointed at a **project-local**
sibling now resolve only in the repo that wrote them, and are dead from every
other consumer of the tier. Strip those pointers as part of the move.

**Why:** mounting the `ddaanet` tier in micro routed 48 facts up. Sixteen of
them carried `[[ghmem-project]]`, `[[micro-colwrap-project]]`,
`[[worktree-handoff-root]]` and similar. Nothing was authored wrong — they were
provenance citations ("**Why:** during ghmem subproject 3
([[ghmem-project]])", "confirmed twice on ghmem's `sync.py`") and they resolved
correctly while everything lived in one flat directory. The *move* created the
boundary underneath prose that predated it. This is not the blessed
write-it-later dangling link: the target is deliberately staying local, so the
link never becomes valid, and every project accumulates its own local facts
that are at best tangentially relevant to the others.

**How to apply:** when routing a fact up, strip the `[[...]]` and keep the
prose that names the incident — "confirmed twice on ghmem's `sync.py`" is
evidence and belongs in the tier; the pointer is the only part that crossed the
boundary. Where the link *was* the whole reference ("Related:
[[some-project]]"), delete it; a tier fact stands on its own Why and How-to-
apply or it is not a tier fact. Watch for two adjacent cases: a link naming a
*skill* rather than a memory, and one whose target the prose already names, so
the pointer is pure redundancy. See [[memory-writing]] and
[[tier-routing-plugin-shaped]].
