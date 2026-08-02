---
name: tests-must-go-red
description: "what does not count as a red (an error, green-at-first); a \"stays silent\" negative passing vacuously, the negative/positive pairing over one fixture, testing a wait-then-proceed gate; an untestable negative is deleted whole, not trimmed back"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: ef3c82af-f6b0-46b1-a5ed-150fdb5b8a25
  modified: 2026-08-01T14:28:21.730Z
---

An assertion counts as evidence only once it has been observed failing. An
*error* is not a red: a run that dies on a missing file, a parse error, or a
crash never reached the assertion, so it proves the harness executes and
nothing more. An assertion that was green the first time it ran is not
evidence either — it has never demonstrated that it discriminates.

Where ordering can buy a genuine red for free, take it: write the check before
the thing it checks. Where it cannot — a test pinning behaviour that already
works — the answer is **structural**, not a separate validation pass. Build the
assertion so that it cannot pass while the thing it watches is absent.

## The three shapes that pass vacuously

- *A negative assertion.* A test that asserts something does not happen passes
  before the feature exists, and keeps passing if the guard it protects is
  later deleted. It contributes to the green count while proving nothing.
- *A positive over an accumulating channel.* A log, a status line, a
  notification payload several producers append to. `assert_output --partial
  "$path"` passes with the new message deleted, because some other notice names
  that path too. Match a phrase unique to the message under test, never an
  identifier the channel carries for other reasons.
- *A negative against a fall-through gate.* This one cannot pass at all:
  asserting a guarded action did not happen, when the gate guarding it waits
  and then proceeds anyway. Given enough wall clock the action always happens —
  that is what the fall-through is for, since never acting is worse than acting
  late. It fails immediately and reads as a bug in working code, so the reflex
  is to "fix" the implementation, or to weaken the test until it passes and
  thereby stop testing the gate.

## Structure them so they cannot go vacuous

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
- *A fall-through gate* — the gate buys **latency**, so latency is the
  assertion: with the condition held false, the action must be delayed by
  roughly the timeout, and removing the gate collapses that gap to zero. Read
  what the gate does when its condition never becomes true before writing the
  test: falls through ⟹ assert the delay; aborts or records a failure ⟹ assert
  *that*. Only a gate that blocks forever is tested by absence, and one of
  those is usually a bug.

Few grouped positives license many negatives: one positive over a fixture pins
the strings and proves the path, and each negative flips a single condition
against it. That licenses the *coverage*, not the evidence. A test body runs
under errexit, so the grouped positive aborts the test the moment it fails and
every negative after it is never reached: a negative sitting behind a positive
runs only in the case where the positive already held. Give each negative its
own test body, or place it ahead of the positive, so its result is its own.

**Where the absent string is a variant of the present one, drop the pair and
assert the exact block.** A prefixed twin of an unprefixed line, the superseded
wording of a line whose new wording is asserted: no single fault produces both,
so the negative restates the positive and nothing can fail it alone. An
exact-block equality — the file's whole bullet region, in order — has no absent
string to go stale, and a dropped line, a stray duplicate, a reorder and a
wording drift each fail it. Same move for "this file was not written at all":
byte-compare against a copy taken before the pass, rather than grepping for the
mark a write would have left. A write that reordered without leaving the mark is
just as much a write.

**Check that the observable the negative watches is one the positive path
actually produces.** A "nothing happened" proxy that is equally true on the
success path pins nothing at all — a hook asserted not to write a settings file
that its own design forbids writing in either branch, a marker no branch
creates. This is a third staleness, beside string and path: the *proxy* never
discriminated, so no change anywhere can fail it. Pick the observable from what
the feature emits when it does fire — the config keys it sets, the files it
installs, the bytes on its output channel — and assert their absence.

**Where a fallback can supply the same answer as the rule, order the fixture so
it cannot.** A preference implemented as "use the matching entry, else the first
one" is not pinned by input whose first entry is the matching one: deleting the
preference leaves the test green. Put the decoy first. Same shape as the
fixture-not-comment rule below, one level in: the fixture has to make the two
implementations disagree, not merely exercise both.

**A pairing is a property of the fixture, not of the comment above it.** A
negative whose fixture differs from its positive in more than the guard's
trigger input is unpaired however confidently the comment claims otherwise: it
can bail out earlier, on a condition neither test names, and pass with the
guard deleted. The tell is a fixture assembled for convenience — reusing the
positive's setup and moving the *subject* rather than the trigger. Building one
that reaches the guard often means constructing the odd layout the guard
actually defends against, which is also the fastest way to find out whether the
guard is load-bearing at all, or what its real justification is.

Where a feature writes to two channels, pin each independently: dropping one
must fail its own assertion and leave the other's green, or the two are not
separately pinned.

Where a negative genuinely cannot fail — it refutes wording no producer emits
at all, so the property is untestable by design — delete the whole test rather
than trimming it back to its testable residue. A title stating an untestable
claim is the thing to cut; keeping the assertions that survive leaves a test
whose name no longer describes it.

A negative refuting a **third-party** tool's message (git's "embedded git
repository" advice, a compiler warning) still pairs: run that tool's producing
path directly over the same fixture, so one assertion proves the message is
still live in this version and reaches this store, while the other proves your
own code avoids it.

**Why:** Writing a negative case first and seeing it green feels like
red-green, but the red never happened. The suite then reports coverage of a
branch no test can distinguish from its absence — the same failure
[[test-the-invocation-path]] names one level up, where a whole suite never ran.
The structural answers above cost no separate pass and keep working as the code
moves, which is what makes them the answer: a validation ritual performed once
by hand is a test run by hand ([[no-handrun-tests]]), and it stops being true
the moment the code changes.

Related: [[bats-shellcheck-gotchas]] — the specific bats mechanisms
(`run missing_function` exits 127 before the code exists; `run` is a subshell
so out-params never reach the test) that make these failure modes concrete in
that framework; [[tests-that-cannot-discriminate]] for the assertion and fixture
shapes that pass whatever the code does.
