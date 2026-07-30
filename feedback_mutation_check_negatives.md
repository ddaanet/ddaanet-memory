---
name: feedback_mutation_check_negatives
description: "a \"stays silent\" negative, or a positive match on a shared output channel, passes vacuously; mutate the guard and watch it go red before counting it; a wait-then-proceed gate is tested by DELAY, never by absence"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a133f610-ff3a-43b5-8aca-054482eb372f
  modified: 2026-07-30T17:35:07.739Z
---

A test that asserts something *does not* happen passes before the feature
exists, and keeps passing if the guard it is meant to protect is later
deleted. It contributes to the green count while proving nothing.

A positive assertion is vacuous in one case: when it matches against an
**accumulating output channel** — a log, a status line, a notification
payload several producers append to — and the string it matches is one an
unrelated producer already emits. `assert_output --partial "$path"` passes
with the new message deleted, because some other notice names that path too.
Match a phrase unique to the message under test, not an identifier the
channel carries for other reasons.

The neighbouring failure is a negative that cannot pass **at all**: asserting
that a guarded action did not happen, when the gate guarding it *waits and then
proceeds anyway*. Given enough wall clock the action always happens — that is
what the fall-through is for, since never acting is worse than acting late. The
gate buys **latency**, so latency is the assertion: with the condition held
false, the action must be delayed by roughly the timeout, and removing the gate
collapses that gap to zero. This one does not hide in a green suite; it fails
immediately and reads as a bug in working code, so the reflex is to "fix" the
implementation or to weaken the test until it passes and thereby stop testing
the gate. Before writing the test, read what the gate does when its condition
never becomes true: falls through ⟹ assert the delay; aborts or records a
failure ⟹ assert *that*. Only a gate that blocks forever is tested by absence,
and one of those is usually a bug. Encountered 2026-07-30 in handoff, where an
approved design specified "make the stub report busy and check the second line
is not sent" for a re-idle gate that falls through by design — the correction
belonged in the plan as well as the test.

**Why:** Writing the negative case first and seeing it green feels like TDD
red-green, but the red never happened. The suite then reports coverage of a
branch no test can distinguish from its absence, which is the same failure
[[feedback_test_the_invocation_path]] names one level up — there, a suite that
never ran; here, an assertion that cannot fail.

**How to apply:** After the feature is green, mutate the specific guard the
assertion watches — widen the condition it excludes, delete the early return,
reorder the call past its bail, drop the one message the test reads — re-run,
and confirm *that* test goes red. Restore and re-run. One targeted mutation
per assertion, not a general mutation sweep; the point is to name which line
the test is watching. Where a feature writes to two channels, mutate each
independently: dropping one must fail its own assertion and leave the other's
green, or the two are not separately pinned.
Cheap enough to do inline (copy the file, patch, run, restore) and it belongs
in the same pass as writing the test, not a later audit.

Related: [[reference-bats-shellcheck-gotchas]] — the specific bats mechanism
(`run missing_function` exits 127, satisfying `-ne 0` before the code exists)
that makes this failure mode concrete in that framework.
