---
name: remove-cleanly-no-vestigial
description: "cut all machinery in one pass; no absence-tests, no vestigial fixtures, inline a collapsed script; if a design proved harmful the cleanup is total"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 77fc3f96-023a-482a-be11-301d61bd7f4a
---

When you cut a capability, cut all of it in the same pass. Do not leave
behind: tests that assert the *absence* of the removed behavior ("test
that it's not what it used to be"), fixtures or sentinel data that stood
for the removed inputs, or a now-trivial script preserved as its own
file/subprocess. A module collapsed to a few lines should be inlined
into its single caller, not kept as a hollow shell.

**Why:** half-removals force the reader to reconstruct what's live vs.
vestigial, and absence-guards grow stale defending behavior nobody will
re-add — pure noise. If a design "proved harmful," the cleanup is total,
not a museum of guards against its return.

**How to apply:** after removing a feature, grep for its whole machinery
— helpers, constants, fixtures, tests, docs, recipes — and cut each;
convert any absence-guard into the positive contract of what remains.
Shares the spirit of [[no-compat-aliases]] (no compat shims)
and [[design-doc-writing]] (the docs analogue).
