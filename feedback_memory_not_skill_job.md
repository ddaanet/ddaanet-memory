---
name: feedback_memory_not_skill_job
description: "a memory instructing what a skill already does is a category error — delete it, don't refine it"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: b2e74378-5da1-458d-8e13-01ab5f03eebe
  modified: 2026-07-26T19:02:21.283Z
---

A memory that tells the agent how to operate a skill is in the wrong place.
The skill carries its own instructions and loads them at its own trigger; a
memory restating them is a second copy that drifts, and it pays index bytes
every session to say what the skill says at the moment it matters.

**Why:** David, 2026-07-26, deleting two gitlore memories that said to write
`.claude/handoff*.md` only through the handoff skill and to fold them into the
code commit — "those two memories are trying to do the job of the handoff
skill. Category error." Both were accurate. Accuracy is not the test.

**How to apply:** when a memory's body reduces to "when using <skill>, do X",
delete it and let the skill say X. If X is missing from the skill, fix the
skill. What legitimately stays in memory is the fact the skill cannot know:
a correction the user made, a failure mode observed in this repo, a constraint
from outside the skill's world. Same test as
[[feedback_no_speculative_rules]] — no incident, no entry — applied to
ownership rather than evidence.
