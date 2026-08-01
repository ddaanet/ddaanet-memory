---
name: mutate-code-to-test-the-tests
description: "proving a test has teeth, and what an unkillable mutation means"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: d81694ba-c563-49a4-a0ce-f8029e7ca7fd
  modified: 2026-07-20T21:20:23.247Z
---

During review, do not judge a test by reading it. Delete or invert the
production code it claims to cover, re-run, and see whether it fails.
Restore afterwards and re-run the full gate to prove the tree is
clean.

Two failure shapes this catches that reading misses:

- **Assertions satisfied by birth state.** A test asserting
  `topic.dirty is True` after a re-route passes even with the entire
  old-topic-dirtying branch deleted, because the topic was created
  dirty and nothing ever clears it.
- **Algebraic tautologies.** An invariant like
  `routed + no_signal + pending + dispatched == total` cannot fail when
  `routed + no_signal` is `triaged` by construction and `pending` is
  *defined* as `total - triaged - dispatched`. It holds at
  `pending = -2`.

- **Vacuous equality between two derived states.** "Doing it live
  equals replaying it" passed against a do-nothing stub, because two
  empty mirrors compare equal. Any test whose assertion is
  `derived_a == derived_b` needs a separate non-emptiness guard, or it
  proves only that both paths did nothing. Caught because the method
  was STUBBED before the test was written per
  [[genuine-red-not-missing-sut]] — against a finished
  implementation it would have gone green and been believed.

- **Equal expected values hide a swap.** Two counts both asserted as
  `1` cannot detect their predicates being exchanged. Whenever a test
  pins several parallel numbers, make them mutually distinct — the
  fixture, not the assertion, is what carries the discrimination.
- **Nothing to leak means nothing to catch.** A correlated-subquery
  mutation (dropping `.correlate()`) survived because the fixture gave
  only ONE entity the data the cross-join would have pulled in. Give a
  second entity its own data and the same mutation dies. Applies to
  any isolation claim: to test that A does not see B's rows, B must
  HAVE rows.
- **An idempotence test never proves a teardown step.** "Rebuild twice,
  get the same rows" stays green with the schema-drop deleted, because
  every write is an upsert. Only a row that must VANISH — inserted out
  of band, then rebuilt away — proves the teardown runs.
- **A substring assertion is satisfied by the wrong line.** Asserting
  `"topic-a" in output` passed with the entire dirty-set line deleted,
  because another line also mentions it. Assert the line, not the
  token.

Related smell: a read path tested against hand-authored fixtures that
the write path is incapable of producing. Check that fixtures are
reachable from the real pipeline, not just well-formed.

**A mutation that cannot be killed may be diagnosing a missing seam,
not an untestable behaviour.** Deleting ghmem's old-topic dirtying
survived every replay test, and no fixture could kill it: replay starts
from a dropped schema, so the topic an item leaves was always created
dirty by the route that put it there. The behaviour was only reachable
from a live apply path that had no public entry point. Opening that
seam killed the mutation immediately. So when a mutation looks
structurally unprovable, ask which entry point is missing before
concluding the behaviour cannot be tested — and until it exists, rename
the test to what it actually proves and say the gap out loud in its
docstring. Never leave a test whose NAME claims coverage that mutation
has shown it does not have.

**Why:** the ghmem Part A eyeball review found three defects this way
that four prior reviews (and reading the tests) had missed.

**How to apply:** for each test guarding non-trivial logic, name the
mutation that should break it. If you cannot name one, the test is
restating the fixture rather than pinning behavior.

See [[genuine-red-not-missing-sut]].
