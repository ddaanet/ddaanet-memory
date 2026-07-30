---
name: feedback_memory_writing
description: "How to write a memory — state what is true for a future reader, not what happened to the writer"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: de3d2151-a51d-4e13-8bd4-cee0e4c48775
  modified: 2026-07-30T13:45:49.118Z
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

**How to apply:** before saving, ask "what is the new learning?" and keep only
the answer. Lead with the rule in the present tense; give the reasoning that
makes it decidable in a new situation; cut every sentence that is narration
rather than instruction. If nothing survives the question, there was no
memory to write — an empty flush is the normal case.

Write and Edit reach the memory directory normally, but the Bash sandbox sees
it as a read-only filesystem: removing a memory file needs
`dangerouslyDisableSandbox`, or `rm` fails with "Read-only file system".

Related: [[feedback_memory_not_other_owners_job]] states the last habit at
length — which of a skill, a design doc, a code comment or CLAUDE.md owns the
lesson instead — and asks the same "what is the new learning?" question;
[[feedback_design_doc_writing]] states the present-tense rule and the
future-reader vantage as they apply to docs.
