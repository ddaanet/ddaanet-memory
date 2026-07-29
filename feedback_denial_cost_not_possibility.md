---
name: feedback_denial_cost_not_possibility
description: "hook path when a denial strands something unrepeatable (an approved summary); direct agent call when it costs one turn"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 9fbebf8e-9dd1-4586-b541-3a6c5ca430a0
  modified: 2026-07-29T18:10:53.028Z
---

A plugin action goes through an intent file plus a hook when a denial would
strand something unrepeatable — a user approval, an ordering guarantee. It goes
through a direct agent call when a denial costs a turn.

gitlore's standalone memory commit takes the hook path: a denial there strands a
summary the user already approved, and an approval cannot be replayed. Its push
takes the direct call: nothing is approved at push time, and a denied push moves
no ref.

**Why:** the hook path costs a second IPC file, a second hook, a stranded-trigger
check, and a second place the flow has to be read from. That is insurance, and it
is priced against the loss it covers.

**How to apply:** name the unrepeatable thing the action carries. If there is
none, let the agent make the call and handle a denial as an ordinary failure
path. Whether a denial is *possible* is not the criterion.

Related: [[feedback_classifier_denied_self_config]] (which actions draw a denial),
[[feedback_harness_over_agent]] (the harness owns detection and branching either
way — this is about who invokes, not who decides).
