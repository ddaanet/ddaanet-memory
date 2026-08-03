---
name: genuine-red-not-missing-sut
description: "TDD red = failed assertion (stub-then-assert), never a missing module, and green-at-first is not evidence; one inert-stub run reds a whole batch but proves absence not wrongness; plan interfaces one contract per line"
metadata:
  node_type: memory
  type: feedback
  originSessionId: bf438d98-adf7-40de-8182-d24d8cf0c199
  modified: 2026-08-03T10:15:46.464Z
---

Two plan-quality rules my human partner enforced on the ghmem
corpus-store plan.

**Genuine red.** A TDD "watch it fail" step must fail on a **failed
assertion**, never because the module/command doesn't exist yet.
"Run both, verify they fail (module/command missing)" is not a
behavioral test — an ImportError proves nothing about whether the test
asserts anything. The correct step: stub the SUT importable but inert
(no-op / deliberately-wrong return), run, and confirm each test fails
on its assertion (wrong value, absent raise), then implement to green.

An assertion that was green the first time it ran is not evidence
either — it has never demonstrated that it discriminates, and no
reviewer reading the finished test can recover that history from the
artifact.

**Why:** "Never accept 'missing SUT' in place of 'failed assertion'."
A test that only errors on a missing symbol hasn't been shown to detect
wrong behavior.

**The inert stub is what makes a batch of tests cheap to validate.**
Run N tests against it once and every test's red is individually
visible in the output; a test that comes back green has identified
itself as vacuous. This is why the ordering matters — tests written
before there is anything to pass them get their red for free, whereas
tests validated against a finished implementation have to manufacture
the absence the ordering would have given, per test. What an inert stub
does *not* prove is that a test detects wrongness rather than absence:
an assertion can fail against nothing and still pass against both the
correct implementation and the plausible-but-forbidden one it exists to
rule out. That case needs the wrong implementation run against the
batch — a second run, not a per-test exercise. Shapes to look for while
reading the output: [[green-is-not-evidence]].

**Interface contracts one-per-line.** In a contract-style plan,
conveying a full interface as a run-on paragraph (semicolon-chained
methods, comma-run dataclass lists) is a form of cramming — the second
form beyond long-line line-count gaming ([[honest-line-count-caps]]).
Give each method / dataclass / exception its own bullet with its full
signature and return type. Un-cramming also surfaces elided return
types (caught three missing `list[...]` annotations this way).

**How to apply:** in every plan, phrase red steps as stub-then-assert;
lay out every `Produces` interface one contract per line.
