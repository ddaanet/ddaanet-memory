---
name: guard-reveals-singleton
description: "a finding ending in \"add a mutual check\" means a singleton is modelled as N *named* instances"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 21058348-2f53-48c1-bc94-de47fa810158
  modified: 2026-08-11T18:10:34.263Z
---

When a finding ends in *add a guard so both cannot fire*, treat that as a
symptom and ask what makes the collision representable at all. The usual
answer: a singleton resource is modelled as N parallel instances with each
instance's identity encoded in its **name** — filename, hook name, script
name — so the at-most-one invariant has nowhere to live. Moving the identity
into the data (one file whose body says which kind it is) makes the collision
unrepresentable and collapses the pipeline that was duplicated per instance.

**Why:** my human partner, 2026-07-30. A design review reported two `Stop`
hooks racing for one tmux composer and recommended a mutual check between
them; the reply was that this sounded like a symptom of a deeper design
issue. It was: the armed transition's identity lived in its *filename*, so
three near-identical pipelines existed and the at-most-one invariant had
nowhere to live. A guard is cheap to propose and it conceals the duplication
that made it necessary, so a review that stops there ships N pipelines plus a
check instead of one pipeline.

**How to apply:** treat your own "add a check" recommendation as the trigger.
Count what else is duplicated per instance — constants, validators, entry
points, failure channels, stale-state sweeps. More than one or two and the
merge is the finding; the guard is not. Sibling-over-parameterisation is
often right at two instances and stops paying at three, and the merge means
deleting the siblings outright ([[remove-cleanly-no-vestigial]]),
not leaving shims.
