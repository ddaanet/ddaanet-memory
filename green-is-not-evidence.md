---
name: green-is-not-evidence
description: "accepting a green test you did not watch fail, yours or inherited: shapes that pass whatever the code does (birth state, algebraic tautology, empty derived states compared, parallel counts all equal, same-second twin fixture commits are one object, an isolation fixture with nothing to leak, upsert idempotence for teardown, a substring another line satisfies, a downstream effect standing in for the write, the right exception from the wrong guard); structuring a negative against string/path/proxy staleness, the negative/positive pairing over one fixture, a wait-then-proceed gate, and deleting an untestable negative whole"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: df6d1dc0-1862-40d1-ae3b-48ec59dd93a3
  modified: 2026-08-13T19:23:43.103Z
---

A test is evidence only when there is a state of the world in which it fails.
The shapes below have none — each asserts something true of the *fixture*
rather than of the code, so it stays green with the behaviour it names removed.
The fix in every case is the same move: make the fixture carry the
discrimination, so the assertion has something to be wrong about.

Green is not the bar; green *for the stated reason* is. What counts as having
watched an assertion fail — and how to get that failure for a whole batch of
tests at once — is [[genuine-red-not-missing-sut]].

## Shapes that pass whatever the code does

- **Satisfied by birth state.** `topic.dirty is True` after a re-route holds
  with the whole dirtying branch gone, because the topic was created dirty and
  nothing clears it. Assert a transition, or start from the state the code must
  change.
- **Algebraic tautology.** `routed + no_signal + pending + dispatched == total`
  cannot fail when `routed + no_signal` *is* `triaged` by construction and
  `pending` is defined as `total - triaged - dispatched`. It holds at
  `pending = -2`. Pin the components against independently known numbers.
- **Two derived states compared.** "Doing it live equals replaying it" is
  `() == ()` when both sides are empty in the scenario, which is vacuously true
  whether or not the reducer agrees with itself. Any `derived_a == derived_b`
  needs a separate non-emptiness guard, or it proves only that both paths did
  nothing.
- **Parallel values all equal.** Two counts both asserted as `1` cannot detect
  their predicates being exchanged. Where a test pins several parallel numbers,
  make them mutually distinct.
- **Deterministic twins meant to be strangers.** Two "unrelated" fixture git
  repos each seeded with an empty commit under the same test identity and
  message are the *same object* whenever setup lands both in one clock second —
  commits have no randomness. Any surface that needs the two sides to differ (a
  consumer's submodule at the same path as a foreign gitlink, so the fetch must
  fail to resolve one against the other) silently vanishes: the "missing"
  commit is already present, and the test passes vacuously on fast runs and
  exercises the collision only when setup straddled a second boundary. Seed
  distinct content or messages so the twins can never coincide.
- **An isolation fixture with nothing to leak.** To prove A does not see B's
  rows, B must HAVE rows. A cross-join or a dropped `.correlate()` is invisible
  when only one entity carries the data the leak would pull in.
- **Idempotence standing in for teardown.** "Rebuild twice, get the same rows"
  stays green with the schema drop deleted, because every write is an upsert.
  Only a row that must VANISH — inserted out of band, then rebuilt away —
  proves the teardown runs.
- **A substring another line satisfies.** `"topic-a" in output` passes with the
  entire dirty-set line deleted, because a different line also names it. The
  same defect in a channel several producers append to — a log, a status line,
  a notification payload: `assert_output --partial "$path"` passes with the new
  message deleted, because some other notice names that path too. Assert the
  line, not the token, and match a phrase unique to the message under test,
  never an identifier the channel carries for other reasons.
- **A downstream effect standing in for the write.** A `record_verdict` test
  called `serve_next` afterwards and asserted the next item was served, which
  "proved" the verdict was recorded — but the queue advances on the `Dispatch`
  row `serve_next` itself wrote. Deleting the entire `record_routing_event`
  call left all ten tests green: the write was **completely unobserved**. Where
  a test observes a downstream effect, assert the actual write directly —
  `store.routes()`, the ledger's last event.
- **The right exception from the wrong guard.** Two note-validation tests
  raised the right `TriageSessionError`, from the dispatch-ownership guard
  firing first rather than the note check they were named for. When a test
  pins an exception, check nothing upstream raises the same type.
- **A bare negative assertion.** A test asserting something does not happen
  passes before the feature exists, and keeps passing if the guard it protects
  is later deleted. It contributes to the green count while proving nothing.
- **A negative against a fall-through gate.** This one cannot pass at all:
  asserting a guarded action did not happen, when the gate guarding it waits
  and then proceeds anyway. Given enough wall clock the action always happens —
  that is what the fall-through is for, since never acting is worse than acting
  late. It fails immediately and reads as a bug in working code, so the reflex
  is to "fix" the implementation, or to weaken the test until it passes and
  thereby stop testing the gate.

Related smell: a read path tested against hand-authored fixtures the write path
cannot produce. Check that fixtures are reachable from the real pipeline, not
merely well-formed.

## Structure a negative so it cannot go vacuous

Where ordering can buy a genuine red for free, take it: write the check before
the thing it checks. Where it cannot — a test pinning behaviour that already
works — the answer is **structural**, not a separate validation pass. Build the
assertion so that it cannot pass while the thing it watches is absent.

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

**Why:** across the five tasks of ghmem dispatch B1 (2026-07-21), every single
task shipped at least one test that passed for the wrong reason, and reading
the tests found none of them — every assertion on the page is correct, and the
wrong cause lives in the fixture, which is the part review skims. A test that
sets up state and asserts an outcome is satisfied by any cause producing that
outcome, and in a system with several coupled writes (a ledger, a mirror, a
queue that filters on a different row) there is usually more than one. Writing
a negative case first and seeing it green feels like red-green, but the red
never happened; the suite then reports coverage of a branch no test can
distinguish from its absence — the same failure [[test-the-invocation-path]]
names one level up, where a whole suite never ran. The structural answers above
cost no separate pass and keep working as the code moves, which is what makes
them the answer: a validation ritual performed once by hand is a test run by
hand — behaviour worth checking belongs in the suite, never in a throwaway
fixture — and it stops being true the moment the code changes.

**How to apply:** for each test guarding non-trivial logic, name a state of the
world in which it fails, and name what else could produce the observable it
checks. Where anything else can, the fixture has to be rebuilt until nothing
else can. Where a behaviour has no such state because it has no entry point of
its own, that is a missing seam in the code: open the seam rather than
accepting the test, and until it exists, rename the test to what it actually
proves. Never leave a test whose NAME claims coverage it does not have. Apply
this hardest to green you did not write — inherited or subagent-authored work
— where nothing shows you the red that never happened; in subagent-driven
development it is not an occasional risk, so budget one wrong-reason test per
task and let a reviewer a rung above the implementer hunt it — which needs the
red phase dispatched separately from the green one, or nobody ever saw the test
fail. It pairs with the sibling lesson that briefs themselves misdescribe the
code.

Related: [[bats-shellcheck-gotchas]] — the specific bats mechanisms
(`run missing_function` exits 127 before the code exists; `run` is a subshell
so out-params never reach the test) that make these failure modes concrete in
that framework.
