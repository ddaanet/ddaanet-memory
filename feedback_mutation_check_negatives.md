---
name: feedback_mutation_check_negatives
description: "an exemption/\"stays silent\" test passes vacuously against code that doesn't exist; break the guard it protects and watch that test go red before counting it"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a133f610-ff3a-43b5-8aca-054482eb372f
  modified: 2026-07-23T09:35:33.679Z
---

A test that asserts something *does not* happen passes before the feature
exists, and keeps passing if the guard it is meant to protect is later
deleted. It contributes to the green count while proving nothing. Positive
assertions do not have this problem — they fail loudly when the code is
absent — so the risk is concentrated in exemption, suppression and
"stays silent" cases.

**Why:** Writing the negative case first and seeing it green feels like TDD
red-green, but the red never happened. The suite then reports coverage of a
branch no test can distinguish from its absence, which is the same failure
[[feedback_test_the_invocation_path]] names one level up — there, a suite that
never ran; here, an assertion that cannot fail.

**How to apply:** After the feature is green, mutate the specific guard the
negative case protects — widen the condition it excludes, delete the early
return, reorder the call past its bail — re-run, and confirm *that* test goes
red. Restore and re-run. One targeted mutation per negative assertion, not a
general mutation sweep; the point is to name which line the test is watching.
Cheap enough to do inline (copy the file, patch, run, restore) and it belongs
in the same pass as writing the test, not a later audit.
