---
name: feedback_mutation_check_negatives
description: "a \"stays silent\" negative, or a positive match on a shared output channel, passes vacuously; hold trigger strings in test-side globals and pair each negative with a positive over the same fixture — positive red = path/wording drift, negative red = guard gone; mutate each guard, a pairing claim isn't a pairing; a wait-then-proceed gate is tested by DELAY, never by absence"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a133f610-ff3a-43b5-8aca-054482eb372f
  modified: 2026-07-31T17:30:49.853Z
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

**Structure negatives so they cannot go vacuous.** Two failure modes, two
structural answers:

- *String staleness* — the negative asserts the absence of wording production
  no longer emits. Hold every trigger string in a global defined in the test
  helper, and assert it from a positive test as well as the negative. A rename
  in production turns the positive red, the global is corrected once, and the
  negative moves with it. Define the literal test-side: sourcing it from the
  production lib moves both sides together and the positive stops pinning the
  wording.
- *Path staleness* — the negative passes because its fixture never reaches the
  guard at all. Pair each negative with a positive over the **same fixture**,
  differing only in the guard's trigger input. Positive red means the path or
  the wording drifted; negative red means the guard was deleted or widened.

Few grouped positives license many negatives: one positive over a fixture pins
the strings and proves the path, and each negative flips a single condition
against it.

**A pairing is a property of the fixture, not of the comment above it.** A
negative whose fixture differs from its positive in more than the guard's
trigger input is unpaired however confidently the comment claims otherwise: it
can bail out earlier, on a condition neither test names, and pass with the guard
deleted. The tell is a fixture assembled for convenience — reusing the positive's
setup and moving the *subject* rather than the trigger. Building one that reaches
the guard often means constructing the odd layout the guard actually defends
against, which is also the fastest way to find out whether the guard is
load-bearing at all, or what its real justification is.

**Why:** Writing the negative case first and seeing it green feels like TDD
red-green, but the red never happened. The suite then reports coverage of a
branch no test can distinguish from its absence, which is the same failure
[[feedback_test_the_invocation_path]] names one level up — there, a suite that
never ran; here, an assertion that cannot fail.

**How to apply:** default to the paired structure above — it costs no separate
pass and keeps working as the code moves, then mutate each guard once to
confirm the pairing is real, because nothing else distinguishes a paired
negative from one that merely looks paired. Mutation is also the whole cover
where a negative cannot be paired with a positive that proves its path: mutate
the specific guard the assertion watches — widen the condition it excludes, delete
the early return, reorder the call past its bail, drop the one message the test
reads — re-run, and confirm *that* test goes red. Restore and re-run. One
targeted mutation per assertion, not a general mutation sweep; the point is to
name which line the test is watching. Where a feature writes to two channels,
pin each independently: dropping one must fail its own assertion and leave the
other's green, or the two are not separately pinned. Mutation is cheap enough
to do inline (copy the file, patch, run, restore) and belongs in the same pass
as writing the test, not a later audit.

Related: [[reference-bats-shellcheck-gotchas]] — the specific bats mechanism
(`run missing_function` exits 127, satisfying `-ne 0` before the code exists)
that makes this failure mode concrete in that framework;
[[feedback_tests_must_go_red]] for the general rule this specialises.
