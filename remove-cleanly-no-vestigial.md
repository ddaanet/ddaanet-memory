---
name: remove-cleanly-no-vestigial
description: "how much machinery goes in one pass, and what not to leave behind"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 77fc3f96-023a-482a-be11-301d61bd7f4a
  modified: 2026-08-03T13:27:16.516Z
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
— helpers, constants, fixtures, tests, docs, recipes, and inbound
references — and cut each; convert any absence-guard into the positive
contract of what remains. Retiring memory files is the same pass: `rm`
leaves every surviving `[[link]]` to them pointing at content nobody will
write, so sweep the corpus for inbound links too. Where the link carried
meaning, restate the rule inline rather than dropping the sentence —
retiring 24 files cost 40 such rewrites across 30 survivors.
Shares the spirit of the standing no-compat-shims rule
and [[design-doc-writing]] (the docs analogue).
