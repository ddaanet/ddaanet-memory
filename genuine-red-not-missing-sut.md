---
name: genuine-red-not-missing-sut
description: "TDD red = failed assertion (stub-then-assert), never a missing module; plan interfaces one contract per line"
metadata:
  node_type: memory
  type: feedback
  originSessionId: bf438d98-adf7-40de-8182-d24d8cf0c199
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

**Why:** "Never accept 'missing SUT' in place of 'failed assertion'."
A test that only errors on a missing symbol hasn't been shown to detect
wrong behavior.

**Interface contracts one-per-line.** In a contract-style plan,
conveying a full interface as a run-on paragraph (semicolon-chained
methods, comma-run dataclass lists) is a form of cramming — the second
form beyond long-line line-count gaming ([[honest-line-count-caps]]).
Give each method / dataclass / exception its own bullet with its full
signature and return type. Un-cramming also surfaces elided return
types (caught three missing `list[...]` annotations this way).

**How to apply:** in every plan, phrase red steps as stub-then-assert;
lay out every `Produces` interface one contract per line.
