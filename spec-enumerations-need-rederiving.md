---
name: spec-enumerations-need-rederiving
description: "a spec that hand-lists the call sites a change must touch: grep the pattern before reading the list, since conformance passes clean while the enumeration is short"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 019f04cd-0e8c-4f1e-a69a-cd0d94bf75ff
  modified: 2026-09-02T22:12:20.198Z
---

When a design hand-enumerates the sites a change must touch ("eight files carry
this paragraph"), re-derive that list mechanically at review time before
checking conformance.

**Why:** conformance and completeness are different questions, and a
hand-written enumeration answers only the first. A rewire whose spec listed 8
call sites passed conformance exactly — every listed file changed, nothing
outside touched — while a grep at review found 15. The 7 unlisted sites kept
both the duplication the change existed to remove and the dependency it existed
to drop. Every review layer above the grep was blind to this: the implementer
matched the spec, and the spec was the thing that was wrong.

**How to apply:** grep for the pattern the change replaces, not for the files
the spec names. Do it before reading any of them, so the spec's list cannot
anchor the search. A count that disagrees with the spec is a finding against
the spec, filed as completeness rather than conformance — the delivered work
is not at fault and re-reviewing it will not surface the gap.

The same split applies to a phrase that is quietly a claim about the corpus:
"the only place X appears", "these are the N consumers". Treat each as a query
to run, not a fact to carry forward. See [[examine-evidence-drift]].
