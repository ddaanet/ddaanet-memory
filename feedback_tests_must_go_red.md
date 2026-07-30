---
name: feedback-tests-must-go-red
description: A test is only trusted once observed failing on its assertion; validate error-at-first or green-at-first tests by mutation
metadata: 
  node_type: memory
  type: feedback
  originSessionId: ef3c82af-f6b0-46b1-a5ed-150fdb5b8a25
  modified: 2026-07-29T12:12:31.965Z
---

Every assertion must be observed failing before it counts as evidence. An
*error* is not a red: a run that dies on a missing file, a parse error, or a
crash never reached the assertion, so it proves the harness executes and
nothing more. A green-at-first assertion is not evidence either.

For any assertion that cannot get a genuine red from ordering alone, validate
it by mutation: break the production code on purpose, confirm the named
assertion fails, revert, confirm green again. One mutation at a time — two live
at once and a surviving assertion no longer identifies itself.

**Why:** Tests written alongside the code they cover (or pinning behaviour that
already works) can be vacuous and still pass forever. Mutation is the only
cheap proof that an assertion discriminates.

**How to apply:** When writing or reviewing a plan with tests, audit each
assertion for whether it ever goes red. Reorder steps to buy a real red for
free where possible — writing a check before the thing it checks. Where that is
impossible, add an explicit mutation table: the one-line edit, and the exact
assertion labels that must appear in the failure output. A missing label means
the assertion is vacuous — fix the assertion, not the mutation.

Related: [[feedback-brief-confirm-option]].
