---
name: memory-writing
description: "what to cut (anecdote, tense, now/here/currently/this session), what to version-stamp, when \"the user\" is a term of art rather than my human partner, and the category errors that mean delete: \"when using <skill>, do X\", logging decisions the design doc owns, or anything that is common sense or one step from an indexed memory; new file vs merge — several memories each seeming to lack a trigger name one moment, duplication is the tell, splitting further or hanging it off an agent prompt both fail; which tier — an operational gotcha to ddaanet vs project memory, the plugin-shaped litmus of whether the mechanism needs this repo or the class of repo, then the relevance test (true anywhere != relevant everywhere; name the other member that MEETS the symptom, since tier lines compete for a capped index); routing a fact up carries its `[[links]]` verbatim and ones aimed at project-local siblings go permanently dead, unlike a write-it-later dangle"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: de3d2151-a51d-4e13-8bd4-cee0e4c48775
  modified: 2026-08-26T19:24:25.373Z
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
place. The owner states it at the moment it matters — a skill loads its
instructions at its own trigger, a living design doc is read before structural
work — while the memory copy drifts and pays index bytes every session to say
the same thing worse.

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
constraint from outside the artifact's world. A quoted correction may name its
source: an attribution credits evidence, which is not the same act as
referring to my human partner by name, and the rule against the latter does
not reach it. Verify coverage before deleting — grep the owner for the
load-bearing terms rather than trusting a pointer that says it is covered. If
nothing survives the question, there was no memory to write — an empty flush
is the normal case. Same test as [[no-speculative-rules]] — no incident, no
entry — applied to ownership rather than evidence.

## A new file, or a section in one that exists

When several memory files each read as "good content, unclear when it fires",
name the moment each one actually fires at before writing a trigger clause into
any of them. Files with distinct HOW content and a single shared WHEN are one
memory that was split by topic, and per-file trigger clauses are false
precision: they route the reader to a third of the material at the moment all
of it applies.

The tell is duplication. A split that follows a real seam does not repeat
itself; one that followed topic headings states the same rule twice in two
files, in near-identical words, because the rule belongs to the moment rather
than to either topic.

Splitting further never helps — every fragment inherits the same trigger.
Neither does hanging the content off a specialist agent's prompt: that couples
a general fact to one consumer and hides it from every other caller.

**Why:** the ddaanet tier carried three test memories that between them stated
the accumulating-channel rule twice and the vacuous `() == ()` comparison
twice. Read as topics they looked like authoring, fixture design and review;
read as moments they were all one — someone is about to accept a green test.
Merging deduplicated ~2.5KB and produced a single catalogue that is complete at
the point of use, where each file alone had been a third of an answer.

**How to apply:** before adding a WHEN to a routing line, state the moment in
one clause. Same clause for two files ⟹ merge and dedup, then spend the
recovered bytes on a hook rich enough to route the whole catalogue. Different
clauses ⟹ the split is real and the trigger clauses are the fix. Check the
residue for anything that fires at a *different* moment and relocate it rather
than dropping it — the merged file inherits one trigger and only one. Merge for
routing, never for headroom: the byte gain is near zero, and a merged line must
still carry every distinctive literal both entries carried
([[index-compaction-triggers]]). Related: [[green-is-not-evidence]].

## Which tier the fact belongs in

When a fact looks project-local at first glance — a sandbox quirk, an
environment gotcha, a harness interaction — check whether it is actually
scoped to *this repo's content* or to *the class of repo this is* (a
Claude Code plugin, a git-hooked project, etc.) before filing it under
project memory.

**Why:** a mechanism that fires for a whole class of repo has as many
consumers as the class has members, and filing it project-local hides it
from all but one. Evidence naming the repo where it surfaced is not
evidence the mechanism depends on that repo — a sandbox EROFS on a
plugin's own active `hooks/hooks.json` reads as repo-specific, but the
harness mounts any live plugin config read-only, so it fires for every
plugin-shaped repo ([[sandbox-effects]]).

**How to apply:** ask "does this fact's *mechanism* depend on something
unique to this repo, or just on this repo being a member of a class
(plugin, hook-managed repo, submodule-based memory store, …)?" If it is the
latter, route to `memory/ddaanet/` (or the matching tier), merged into an
existing topic file if one already exists for that trigger surface —
don't leave a same-fact stub in the project tier "just in case."

**Class membership is necessary, not sufficient — the second test is
relevance.** A mechanism can belong to the class and still have exactly one
consumer, so ask which *other* member will actually meet the symptom, and name
it. Truth is the wrong test: a fact holds in every repo and is still
project-local when only one repo's code can encounter it. Claude Code's
worktree `cwd` and `$CLAUDE_PROJECT_DIR` payload shapes are the harness's
behaviour and hold identically everywhere, yet the symptoms — a
walk-up-from-`cwd` root check finding nothing, a guard deciding against a
deleted cwd — reach only code that resolves a project root from
`$CLAUDE_PROJECT_DIR`, which is one plugin's job description, so they stay in
that plugin's own store. My human partner, 2026-08-26: "it being true anywhere
does not make it relevant everywhere."

**Why the asymmetry favours staying local:** a tier line is loaded by every
session in every mounting repo, and the index is capped — entries past the
loader cutoff are dropped silently, with no warning at either write or read
time ([[gitlore-tier-index-budget]]). Tier lines therefore compete for a
bounded budget instead of accumulating freely, so a weak promotion costs more
than its own bytes: it crowds entries that route better. Demoting later costs a
file move and one index line; a wrong promotion taxes every repo silently.
Being the repo that *discovered* a fact is not an argument for promoting it,
and neither is a prior session having already promoted it. Separate the
*behaviour* from its *encoding*: how a given repo's code handles the mechanism
is always project-local, and usually belongs in that repo's design doc rather
than memory at all. Check whether the portable half is already covered by an existing tier
fact before adding a new one.

## Links when a fact routes up

When a memory is routed from a project's own store up into a shared tier, its
`[[links]]` travel with it verbatim. Any that pointed at a **project-local**
sibling resolve only in the repo that wrote them, and are dead from every
other consumer of the tier. Strip those pointers as part of the move.

**Why:** mounting the `ddaanet` tier in micro routed 48 facts up. Sixteen of
them carried `[[ghmem-project]]`, `[[micro-colwrap-project]]`,
`[[worktree-handoff-root]]` and similar. Nothing was authored wrong — they were
provenance citations ("**Why:** during ghmem subproject 3
`[[ghmem-project]]`", "confirmed twice on ghmem's `sync.py`") and they resolved
correctly while everything lived in one flat directory. The *move* created the
boundary underneath prose that predated it. This is not the blessed
write-it-later dangling link: the target is deliberately staying local, so the
link never becomes valid, and every project accumulates its own local facts
that are at best tangentially relevant to the others.

**How to apply:** when routing a fact up, strip the `[[...]]` and keep the
prose that names the incident — "confirmed twice on ghmem's `sync.py`" is
evidence and belongs in the tier; the pointer is the only part that crossed the
boundary. Where the link *was* the whole reference ("Related:
`[[some-project]]`"), delete it; a tier fact stands on its own Why and How-to-
apply or it is not a tier fact. Watch for two adjacent cases: a link naming a
*skill* rather than a memory, and one whose target the prose already names, so
the pointer is pure redundancy.

## Reviewing a proposed update

A proposed memory update gets one of four verdicts — discard, relocate, merge,
or save as written. Take the questions in order and stop at the first that
settles it.

1. **Is there an incident?** No incident, no entry ([[no-speculative-rules]]).
2. **Does another artifact own it** — a skill, a design doc, the code, a
   CLAUDE.md? Grep that owner for the load-bearing terms before accepting a
   pointer that claims coverage.
3. **Would the reader reach it unaided**, or does an indexed memory imply it in
   one step? Either answer is a discard.
4. **Who reconstructs it?** A script or the harness recomputing it means drop
   and defer; a model inferring it from a paraphrase, with the failure silent,
   means carry it ([[reconstructable-two-categories]]).
5. **What moment does it fire at?** Same clause as a file already in the store
   means merge into that file, not a new one.
6. **Which tier?** Mechanism scoped to the repo, or to the class of repo.
7. **Does the body survive the strip?** Incident out, first person out,
   deictics out, present tense in, an ageing observation version-stamped.
8. **Does the index line route to it?** The distinctive literal a session would
   arrive holding has to be in the line, and being over the byte cap is never a
   reason to omit it — that pressure is a retirement decision, taken separately
   ([[index-compaction-triggers]]).

The fact's diff rides the commit carrying the source change it documents
([[bundle-memory-with-source]]); administering memory with no parent-side
content needs no parent commit at all
([[gitlore-memory-administration-no-parent-commit]]).

Related: [[design-doc-writing]] states the present-tense rule and the
future-reader vantage as they apply to docs;
[[index-compaction-triggers]] governs the index line that points at
the file; [[gitlore-tier-merge-direction]] and [[gitlore-tier-index-budget]]
cover which index a change is authored in and what mounting a populated tier
costs.
