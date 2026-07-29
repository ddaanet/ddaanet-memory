---
name: feedback_memory_not_other_owners_job
description: "what a skill, design doc, code comment or CLAUDE.md already owns is a category error in memory; delete it, accuracy is not the test, and ask what the new learning is once the anecdote is stripped"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: b2e74378-5da1-458d-8e13-01ab5f03eebe
  modified: 2026-07-29T17:13:02.716Z
---

A memory that repeats what another artifact already owns is in the wrong place.
The owner states it at the moment it matters — a skill loads its instructions at
its own trigger, a living design doc is read before structural work — while the
memory copy drifts and pays index bytes every session to say the same thing
worse.

Shapes seen so far:

- **A skill's job.** The body reduces to "when using <skill>, do X". Delete it
  and let the skill say X; if X is missing from the skill, fix the skill.
- **A design doc's job.** A `project` memory that logs design decisions,
  supersessions and dated status. The design doc *is* the memory of the design —
  organized by subject, rationale preserved in dated entries, which is exactly
  the genre a memory must not adopt ([[feedback_design_doc_writing]]: memories
  state present-tense current truth; dated records belong to the doc).
- **The code's job.** A rule about how one call site must behave belongs in a
  comment there, where it is read by whoever edits that line. A memory saying
  the same thing reaches a session that may never open the file.
- **CLAUDE.md's job.** A convention binding on all work in one repo. CLAUDE.md
  is loaded whole every session in that repo and nowhere else — exactly the
  scope such a rule wants.

The test that separates a memory from an anecdote: **what is the new
learning?** Strip the incident and see what is left. A mechanism that holds
wherever it is met is the memory; the story of meeting it is not, however
instructive it felt at the time. If nothing generalizes, the fix already
landed in the code, the doc or the skill, and there is nothing to write.

**Why:** David, 2026-07-26. Two gitlore memories told the agent to write
`.claude/handoff*.md` only through the handoff skill — "those two memories are
trying to do the job of the handoff skill. Category error." Both were accurate;
accuracy is not the test. The same call landed on a 66 KB `project` memory
logging the tiered-memory design: "I don't see why it should not be subsumed by
the living design, it's there precisely for that purpose: record the memory of
the design."

**How to apply:** name the owner before writing. If a skill, a design doc, a
README or the code itself states it at the point of use, don't restate it. What
legitimately stays in memory is what no owner can carry: a correction the user
made, a failure mode observed in this repo, a constraint from outside the
artifact's world. Verify coverage before deleting — grep the owner for the
load-bearing terms rather than trusting a pointer that says it is covered. Same
test as [[feedback_no_speculative_rules]] — no incident, no entry — applied to
ownership rather than evidence.
