---
name: genuine-red-not-missing-sut
description: "TDD red = failed assertion (stub-then-assert), never a missing module, and green-at-first is not evidence; one inert-stub run reds a whole batch but proves absence not wrongness; plan interfaces one contract per line; mutate the SUT in place — a test relocated to a temp dir rebinds `repo_root` from `$0` and reds for the wrong reason"
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

**Manufacture the red by mutating the SUT in place, never by relocating the
test.** A shell test that resolves its own root —
`repo_root="$(cd "$(dirname "$0")/.." && pwd)"` — rebinds when the test file is
copied to a temp directory and repointed at an old copy of the script.
Fixtures, payload paths and `cwd` values then all name the wrong tree, and
cases fail for reasons unrelated to the change under test: a `-e`-versus-`-d`
probe run from a relocated copy reported eight failures where mutating the
script in place reported one, and only the one was signal. This runs the
opposite way from a vacuous green — it manufactures *red*, so a defect that
does not exist clears the "watch it fail" bar and gets a fix written for it.
Save a copy of the script, mutate it where it lives, restore after. A
red-check recipe handed over in a plan or a skill is where this arrives; check
it against the suite's own root resolution before following it.

**Interface contracts one-per-line.** In a contract-style plan,
conveying a full interface as a run-on paragraph (semicolon-chained
methods, comma-run dataclass lists) is a form of cramming — the second
form beyond long-line line-count gaming ([[honest-line-count-caps]]).
Give each method / dataclass / exception its own bullet with its full
signature and return type. Un-cramming also surfaces elided return
types (caught three missing `list[...]` annotations this way).

**How to apply:** in every plan, phrase red steps as stub-then-assert;
lay out every `Produces` interface one contract per line.
