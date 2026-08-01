---
name: tests-must-go-red
description: "what does not count as a red (an error, green-at-first); a \"stays silent\" negative passing vacuously, the negative/positive pairing over one fixture, a wait-then-proceed gate; mutate each guard, and mutate to the plausible wrong implementation"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: ef3c82af-f6b0-46b1-a5ed-150fdb5b8a25
  modified: 2026-08-01T13:46:31.439Z
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
every negative after it is never reached — a mutation that breaks both only ever
demonstrates the positive. Proving a negative discriminates needs a mutation
that leaves its positive green.

**Where the absent string is a variant of the present one, drop the pair and
assert the exact block.** A prefixed twin of an unprefixed line, the superseded
wording of a line whose new wording is asserted: no single fault produces both,
so the negative restates the positive and no mutation can red it alone. An
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
discriminated, so no mutation anywhere can red it. Pick the observable from what
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

## Mutate each guard — a pairing claim isn't a pairing

Nothing else distinguishes a paired negative from one that merely looks paired,
and mutation is the whole cover where a negative cannot be paired with a
positive that proves its path. Mutate the specific guard the assertion watches —
widen the condition it excludes, delete the early return, reorder the call past
its bail, drop the one message the test reads — re-run, and confirm *that* test
goes red. Restore and re-run. One targeted mutation per assertion, not a general
sweep; the point is to name which line the test is watching. It is cheap enough
inline (copy the file, patch, run, restore) and belongs in the same pass as
writing the test, not a later audit ([[mutate-code-to-test-the-tests]]).

**Mutate to the plausible wrong implementation, not to nothing.** Deleting a
bail outright is usually absorbed downstream — an empty variable makes the next
path check fail on its own, a missing file makes the next read return the
default — so the deletion proves only that the code is defensive in layers.
What the assertion is really defending against is the *substitute*: filling the
missing root in from the session cwd instead of bailing, adding a `2>/dev/null`
so an unparseable config reads as "feature off". Write that version and the
assertion reds cleanly. Read the mutated line back before trusting the run: a
harness that checks only *that* the file changed will report a red for a
mutation that mangled the script, and a red from a broken script is the same
false evidence as an error standing in for a red.

Read a surviving mutation before believing it. Three things look identical from
the green count, and only one of them is a bad test:

- *The mutation missed the guard.* The condition being cut is not the one that
  decides the case, so the test never had a chance to notice. Follow the value
  through the code before concluding anything about the test — and if the cut
  condition really is unreachable from every fixture, that is a production
  finding: an unpinned guard, which is worth its own fixture built around the
  odd layout the guard defends against.
- *The negative is over-determined.* Two independent guards each suffice, so no
  single fault reaches the assertion. Break both at once to tell this apart from
  an unfalsifiable assertion; that combined run is a reachability check, not a
  discrimination claim, and the assertion stays honest belt-and-braces.
- *The negative genuinely cannot fail*, per the variant-of-the-positive case
  above. Replace it, do not annotate it — and where it refutes wording no
  producer emits at all, so the property is untestable by design, delete the
  whole test rather than trimming it back to its testable residue. A title
  stating an untestable claim is the thing to cut; keeping the assertions that
  survive leaves a test whose name no longer describes it.

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
that framework; [[unkillable-mutation-means-a-wrong-model]] for the case where
the surviving mutation is a finding about the spec rather than the test, and
[[mutation-sweep-must-survive-a-kill]] for running mutations in bulk without
leaving the tree mutated.
