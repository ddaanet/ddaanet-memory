---
name: tests-that-cannot-discriminate
description: "assertion and fixture shapes that pass whatever the code does: birth state, algebraic tautology, two empty derived states compared, parallel counts all equal, an isolation fixture with nothing to leak, upsert idempotence standing in for teardown, a substring another line satisfies"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 827ef62f-cfc6-4315-866c-0cc2ba9b73da
  modified: 2026-08-01T14:25:00.281Z
---

A test pins behaviour only when there is a state of the world in which it
fails. These shapes have none — each asserts something true of the fixture
rather than of the code, so it stays green with the behaviour it names
removed. The fix in every case is the same move: make the *fixture* carry the
discrimination, so the assertion has something to be wrong about.

- **Satisfied by birth state.** `topic.dirty is True` after a re-route holds
  with the whole dirtying branch gone, because the topic was created dirty and
  nothing clears it. Assert a transition, or start from the state the code must
  change.
- **Algebraic tautology.** `routed + no_signal + pending + dispatched == total`
  cannot fail when `routed + no_signal` *is* `triaged` by construction and
  `pending` is defined as `total - triaged - dispatched`. It holds at
  `pending = -2`. Pin the components against independently known numbers.
- **Two derived states compared.** "Doing it live equals replaying it" is
  `() == ()` against a do-nothing stub. Any `derived_a == derived_b` needs a
  separate non-emptiness guard, or it proves only that both paths did nothing.
- **Parallel values all equal.** Two counts both asserted as `1` cannot detect
  their predicates being exchanged. Where a test pins several parallel numbers,
  make them mutually distinct.
- **An isolation fixture with nothing to leak.** To prove A does not see B's
  rows, B must HAVE rows. A cross-join or a dropped `.correlate()` is invisible
  when only one entity carries the data the leak would pull in.
- **Idempotence standing in for teardown.** "Rebuild twice, get the same rows"
  stays green with the schema drop deleted, because every write is an upsert.
  Only a row that must VANISH — inserted out of band, then rebuilt away —
  proves the teardown runs.
- **A substring another line satisfies.** `"topic-a" in output` passes with the
  entire dirty-set line deleted, because a different line also names it. Assert
  the line, not the token — and where several producers append to one channel,
  match a phrase unique to the message under test.

Related smell: a read path tested against hand-authored fixtures the write path
cannot produce. Check that fixtures are reachable from the real pipeline, not
merely well-formed.

**Why:** each of these was shipped green, by more than one author, and read as
covered. The assertion is true; it is just not true *because* of the code it
names, and nothing about reading it says so — the fixture is where the defect
lives, and the fixture is the part review skims.

**How to apply:** for each test guarding non-trivial logic, name a state of the
world in which it fails. If you cannot name one, it is restating the fixture
and needs rebuilding, not annotating. Where a behaviour has no such state
because it has no entry point of its own, that is a missing seam in the code:
open the seam rather than accepting the test, and until it exists, rename the
test to what it actually proves. Never leave a test whose NAME claims coverage
it does not have. Related: [[tests-must-go-red]],
[[tests-pass-for-the-wrong-reason]], [[genuine-red-not-missing-sut]].
