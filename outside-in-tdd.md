---
name: outside-in-tdd
description: "starting TDD when the architecture is already fixed: e2e first or units first"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 7dd65fa7-001f-45cd-a975-fcab4433bdf5
  modified: 2026-08-12T19:00:55.170Z
---

When the architecture is already settled (e.g., decided in a design doc), start TDD from a red end-to-end test that names the user-facing contract, then let its failure modes drive what to build next. Don't default to bottom-up unit tests first.

**Why:** Bottom-up unit tests can pass while the integration contract is still broken — they verify internal consistency, not the behavior the user cares about. Outside-in red-first tests are black-box by construction (they assert through the boundary), so:
- the test survives refactors of the internals
- the next failure tells you the next file to write — no speculative helpers
- integration risk (fixtures, stubs, env setup) gets exercised on day 1 instead of being deferred

My human partner pushed back on a bottom-up recommendation for Plan 02 with this reasoning. Plan 01 had its e2e test added last — so its passing only confirmed unit-test self-consistency, not the contract.

**How to apply:** When my human partner has a design doc and you're proposing a plan structure, default to outside-in: e2e (red) → fill in by following failures → backfill unit tests at the seams the e2e doesn't reach. Only fall back to bottom-up when the architecture itself is still being discovered. See related: [[loose-generation]] (constrain at the artifact boundary, not mid-stream).
