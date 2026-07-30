---
name: feedback_memory_writing
description: "state what is true for a reader who wasn't there; cut the anecdote, the tense, the pending framing; \"when using <skill>, do X\" or a project memory logging design decisions the living design doc owns is a category error — delete it, accuracy is not the test; ask what the new learning is"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: de3d2151-a51d-4e13-8bd4-cee0e4c48775
  modified: 2026-07-30T16:14:27.263Z
---

A memory states what is true, addressed to a reader who was not there. The
incident that produced it is scaffolding: useful while writing, removed before
saving.

**Why:** a memory is re-read in sessions that share none of its context. Four
habits make one worse than useless:

- *Describing what used to be.* State the rule that holds now. A snapshot of
  which repos have migrated, which version is current, or what a file used to
  contain becomes false without warning and gets acted on anyway.
- *Writing from the author's present.* "I did X and was corrected", "in one
  session two reviewers…" — the reader has no session, no I, and no way to
  tell whether the anecdote still generalises. First person and a narrated
  incident are the tell.
- *Pending or temporal framing.* Anything phrased as still-to-happen — work
  that "needs a fresh session", a migration "not done yet" — reads as
  outstanding forever and invites redoing finished work.
- *Recording what belongs somewhere else.* If the lesson's real home is a
  comment, the code itself, CLAUDE.md, or a change to a skill, put it there.
  A memory duplicating what the repo already records competes with the
  source and loses.

## What another artifact already owns

That last habit is the one that recurs, so it gets stated at length. A memory
repeating what another artifact owns is in the wrong place. The owner states it
at the moment it matters — a skill loads its instructions at its own trigger, a
living design doc is read before structural work — while the memory copy drifts
and pays index bytes every session to say the same thing worse.

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

**Accuracy is not the test.** David, 2026-07-26: two gitlore memories told the
agent to write `.claude/handoff*.md` only through the handoff skill — "those two
memories are trying to do the job of the handoff skill. Category error." Both
were accurate. The same call landed on a 66 KB `project` memory logging the
tiered-memory design: "I don't see why it should not be subsumed by the living
design, it's there precisely for that purpose: record the memory of the design."

**How to apply:** before saving, ask "what is the new learning?" and keep only
the answer. Strip the incident and see what is left: a mechanism that holds
wherever it is met is the memory; the story of meeting it is not, however
instructive it felt at the time. Lead with the rule in the present tense; give
the reasoning that makes it decidable in a new situation; cut every sentence
that is narration rather than instruction. Name the owner before writing — if a
skill, a design doc, a README or the code itself states it at the point of use,
don't restate it. What legitimately stays in memory is what no owner can carry:
a correction the user made, a failure mode observed in this repo, a constraint
from outside the artifact's world. Verify coverage before deleting — grep the
owner for the load-bearing terms rather than trusting a pointer that says it is
covered. If nothing survives the question, there was no memory to write — an
empty flush is the normal case. Same test as [[feedback_no_speculative_rules]]
— no incident, no entry — applied to ownership rather than evidence.

Write and Edit reach the memory directory normally, but the Bash sandbox sees
it as a read-only filesystem: removing a memory file needs
`dangerouslyDisableSandbox`, or `rm` fails with "Read-only file system".

Related: [[feedback_design_doc_writing]] states the present-tense rule and the
future-reader vantage as they apply to docs;
[[feedback_index_compaction_triggers]] governs the index line that points at
the file.
