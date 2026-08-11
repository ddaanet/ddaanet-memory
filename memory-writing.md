---
name: memory-writing
description: "what to cut (anecdote, tense, now/here/currently/this session), what to version-stamp, when \"the user\" is a term of art rather than my human partner, and the category errors that mean delete: \"when using <skill>, do X\", logging decisions the design doc owns, or anything that is common sense or one step from an indexed memory"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: de3d2151-a51d-4e13-8bd4-cee0e4c48775
  modified: 2026-08-11T20:14:02.172Z
---

A memory states what is true, addressed to a reader who was not there. The
incident that produced it is scaffolding: useful while writing, removed before
saving.

**Why:** a memory is re-read in sessions that share none of its context. Five
habits make one worse than useless:

- *Describing what used to be.* State the rule that holds now. A snapshot of
  which repos have migrated, which version is current, or what a file used to
  contain becomes false without warning and gets acted on anyway.
- *Writing from the author's present.* "I did X and was corrected", "in one <!-- hygiene-ok -->
  session two reviewers…" — the reader has no session, no I, and no way to <!-- hygiene-ok -->
  tell whether the anecdote still generalises. First person and a narrated
  incident are the tell.
- *Pending or temporal framing.* Anything phrased as still-to-happen — work
  that "needs a fresh session", a migration "not done yet" — reads as
  outstanding forever and invites redoing finished work.
- *Recording what belongs somewhere else.* If the lesson's real home is a
  comment, the code itself, CLAUDE.md, or a change to a skill, put it there.
  A memory duplicating what the repo already records competes with the
  source and loses.
- *Recording what the reader would have reached anyway.* A fact that restates
  common sense, or that follows in one step from a memory already indexed,
  spends index bytes and reading time telling a competent reader what it
  already had. The tell is that the body's general claim is unarguable and
  only its instance is interesting — which means the instance was the whole
  content, and the instance is the part that does not generalise.

**Write from the reader's vantage, not the writer's.** The reader is a future
session with no access to the one that wrote the fact, arriving at an unknown
moment for an unrelated reason. So the body states what *is*, never what used
to be or what just changed — a fact phrased as a delta is unreadable once the
before-state is gone. Deictic words give the vantage away and none of them
survive the trip: *now*, *here*, *today*, *currently*, *recently*, *this
session*, *fields seen*, *needs a fresh session*. Each is anchored to a moment
the reader is not in. Name the condition instead of the moment — "if no
statusline is configured" rather than "here it writes to X" — and version-stamp
an observation that could age (`CC 2.1.220`) rather than calling it recent.

**"The user" splits two ways.** My human partner is called "my human
partner", but `user` is also a term of art in most of what these facts
describe: a user-visible hook channel, git's `protocol.file.allow=user`, the
end user a contract faces. Rename only where the referent is a person in our
shared history (ran, said, corrected, pushed back) or the agent's own
addressee; leave every role in the described system alone. One file can need
both calls seven lines apart.

## What another artifact already owns

Recording what belongs somewhere else is the habit that recurs, so it gets
stated at length. A memory repeating what another artifact owns is in the wrong
place. The owner states it
at the moment it matters — a skill loads its instructions at its own trigger, a
living design doc is read before structural work — while the memory copy drifts
and pays index bytes every session to say the same thing worse.

Shapes seen so far:

- **A skill's job.** The body reduces to "when using <skill>, do X". Delete it
  and let the skill say X; if X is missing from the skill, fix the skill.
- **A design doc's job.** A `project` memory that logs design decisions,
  supersessions and dated status. The design doc *is* the memory of the design —
  organized by subject, rationale preserved in dated entries, which is exactly
  the genre a memory must not adopt ([[design-doc-writing]]: memories
  state present-tense current truth; dated records belong to the doc).
- **The code's job.** A rule about how one call site must behave belongs in a
  comment there, where it is read by whoever edits that line. A memory saying
  the same thing reaches a session that may never open the file.
- **CLAUDE.md's job.** A convention binding on all work in one repo. CLAUDE.md
  is loaded whole every session in that repo and nowhere else — exactly the
  scope such a rule wants.

**Accuracy is not the test.** My human partner, 2026-07-26: two gitlore
memories told the agent to write `.claude/handoff*.md` only through the handoff
skill — "those two memories are trying to do the job of the handoff skill.
Category error." Both were accurate. The same call landed on a 66 KB `project`
memory logging the tiered-memory design: "I don't see why it should not be <!-- hygiene-ok -->
subsumed by the living design, it's there precisely for that purpose: record
the memory of the design."

**How to apply:** before saving, ask "what is the new learning?" and keep only
the answer. Strip the incident and see what is left: a mechanism that holds
wherever it is met is the memory; the story of meeting it is not, however
instructive it felt at the time. Then ask whether what is left is new *to the
reader*: would an agent with no memory at all reach it unaided, and does an
indexed memory already imply it in one step? Either answer means delete. This
is the test the ownership one does not cover — nothing else owns the fact, it
simply does not need saying — and the one most easily rationalised past,
because a lesson feels earned in proportion to the effort that produced it
rather than to what it adds. Lead with the rule in the present tense; give
the reasoning that makes it decidable in a new situation; cut every sentence
that is narration rather than instruction. Name the owner before writing — if a
skill, a design doc, a README or the code itself states it at the point of use,
don't restate it. What legitimately stays in memory is what no owner can carry:
a correction my human partner made, a failure mode observed in this repo, a
constraint from outside the artifact's world. Verify coverage before
deleting — grep the owner for the load-bearing terms rather than trusting a
pointer that says it is covered. If nothing survives the question, there was
no memory to write — an empty flush is the normal case. Same test as
[[no-speculative-rules]] — no incident, no entry — applied to ownership
rather than evidence.

Related: [[design-doc-writing]] states the present-tense rule and the
future-reader vantage as they apply to docs;
[[index-compaction-triggers]] governs the index line that points at
the file.
