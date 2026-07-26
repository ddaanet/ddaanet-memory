---
name: feedback_mutation_check_negatives
description: "a \"stays silent\" test, or a positive match on a shared output channel, passes vacuously; mutate the guard and watch that test go red before counting it"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a133f610-ff3a-43b5-8aca-054482eb372f
  modified: 2026-07-26T12:12:09.565Z
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
