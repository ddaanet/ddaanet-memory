---
name: unkillable-mutation-means-a-wrong-model
description: "a mutation a plan mandates but that no test can kill is evidence the plan's model of the code is wrong — investigate the premise, never instruct the implementer to tune the fixture until it bites"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a9133ea6-55ff-41b0-a76f-21cad29dbb69
  modified: 2026-07-22T15:09:09.983Z
---

When a plan says "mutate X and a test must die", **verify that the
mutation actually breaks something before mandating it.** A mutation
that cannot be killed is not a weak test. It is usually evidence that
the plan's model of the code is wrong — the mutated line is a no-op,
or it corrects for something that was never there.

Never write the escape hatch "if it survives, tune the fixture until
it bites." That instruction cannot terminate when the premise is
false, and it sends the implementer hunting for a defect in their own
test to explain a defect in the plan.

**Why:** B2's plan told task 3 to subtract one character from the diff
allowance "for the newline joining the body to the marker", and to
prove it by dropping the subtraction and watching a packet overshoot
the budget. The subtraction was never needed — the renderer emits the
body element even when empty, so that separator already sits inside
the measured floor and a packet is `floor + len(body)` exactly at
every cap. Dropping a correction for a cost that does not exist lands
the packet at precisely the budget, which still satisfies
`packet_chars <= budget`. The mutation was unkillable because the
arithmetic it defended was imaginary.

Measuring the real renderer took one probe script and settled it in a
minute; the plan's claim had never been checked against the code at
all.

**How to apply:** treat an unkillable mandated mutation as a finding
about the spec, not the test — go measure the property the mutation
was supposed to defend, and fix whichever of the two is wrong. Then
replace it with a mutation in the direction that genuinely overshoots
(here, *adding* one to the allowance). Where a plan's arithmetic rests
on an invariant of code another task owns, pin that invariant with its
own test in the owning task, so the next reader does not have to
re-derive it. Related: [[mutate-code-to-test-the-tests]],
[[tests-pass-for-the-wrong-reason]],
[[provisional-values-not-provisional-code]].
