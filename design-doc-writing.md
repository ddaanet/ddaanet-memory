---
name: design-doc-writing
description: "the six sections (FR, NFR, architecture, decisions, rejected alternatives, changelog) + the state layer (status stamp, Now, legend, `Where · pinned by`, limitations vs non-goals); dated rationale vs present-tense truth; prose over tables; changelog = design-significant only, git is the full record"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 618e80ab-c1a4-41c0-94c9-07e03fa4af68
  modified: 2026-08-14T12:57:29.879Z
---

A design doc is a living doc, not a one-off spec. Six sections: functional
requirements, non-functional requirements, architecture, design decisions (with
rationale), rejected alternatives, changelog — and the changelog is a
**separate file** the doc points at. Past a certain size that file is only the
index and each change gets its own dated file beside it —
`docs/changelog/<date>-<title>.md`, one line per entry in the index.

Plans and specs go in a root `plans/`; the spec/design doc is a **single living
doc for the project** — `docs/design.md`. The rule is one living doc, not one
filename; check where the project actually keeps it before creating a second.
Its filename is stable and undated, which is what signals it is living — frozen
specs and plans are date-stamped.

**Why:** it captures the *why* alongside the *what*, so future sessions don't
re-litigate settled decisions. The rejected-alternatives section matters most —
it prevents revisiting approaches already ruled out for good reason.

**How to apply:** when updating, add to the changelog and update the relevant
section. Don't flatten or summarize decisions — preserve the rationale. When
asked for a "live" or "living" design doc, do not reach for brainstorming or ask
about scope and format: the format is settled, and it is this one.

## Two documents, two jobs

The living design answers **what, how and why**, in the present tense: what the
system does, how it is built, why it is built that way. The changelog answers
**how we got here**.

**The changelog is scoped to design-significant changes only.** Git history is
already the full changelog and needs no duplicate. What belongs in the file is
what explains why the project is the way it is — decisions reversed, subsystems
built or torn down, requirements added or dropped, a rationale that turned out
false — never a line-by-line account of every edit. An entry earns its place by
answering a question a future reader would otherwise ask of the design doc and
find no answer to.

That split decides where text goes when a doc is cleaned up. Implementation
status ("slice 3-ii done 2026-07-21, 37 cases in …"), `(implemented
2026-07-15)` tags stamped on a mechanism paragraph, and "this retired the two
earlier point-fixes" narratives are all changelog material: in the design they
are noise, and they rot. Dates that *ground a claim* stay wherever the claim is
— "verified against CC 2.1.150", "measured over 76 bullets", "eval over 528
transcripts" — because deleting them turns evidence into assertion.

## The state layer

The six sections say what the system is and why. They do not say what is true
*right now* or how much to trust it, and a snapshot that omits that tells a cold
reader nothing about where the code is, what is live, or what the next action is.
Five additions carry that, and they sit around the six rather than inside them:

1. **A `Status:` line and a `Verified against: <commit> (<date>)` stamp** at the
   top. Every state claim in the document is anchored to that commit, so a
   reader knows what to `git diff` for drift. Point-in-time "Done" rots without
   it.
2. **Now** — the resume pointer: Focus / Next / Do-not-re-litigate. The
   single highest-value section; without it the next action is only inferrable
   from limitations plus changelog.
3. **A status legend** — fixed vocabulary, so "Done" means one thing. `Done` =
   implemented and pinned by a test; `Done (prose)` = instructions, not code and
   not tested; `Partial`; `Planned`.
4. **A `Where · pinned by` column** on the FR and NFR tables, mapping each
   requirement to its code symbol and the test that proves it. This turns the
   document into a navigation index and makes "Done" re-checkable rather than a
   bare claim. *Verify the symbols and tests exist before writing them* — never
   assert an unverified pin.
5. **Limitations** (inherent — *can't*) split from **Non-goals** (deliberate —
   *won't, for now*). Conflating them hides whether an exclusion is a hard wall
   or revisitable scope.

Numbering makes requirements, decisions and limitations individually
referenceable (FR-3, D-12, L-5) so other artifacts — and other sections — can
cite them.

These are facts about the system's current state, so they belong to the design
doc rather than to any behavioral-rules file. They are a different axis from
project instructions, not a leftover bucket.

## Prose over tables

Prose beats a big table for anything with a paragraph's worth of reasoning in a
cell: a rejected-alternatives or decisions table hides the rationale in one long
cell and forces every entry into the same width. Subsections with a bolded
lead-in read the same at a glance and let the argument breathe. Keep tables for
genuinely tabular data — short, uniform, scannable cells, which is what FR/NFR
rows and transformation contracts are.

Each decision carries its rationale and, where one applies, a **Supersedes** or
**Reopen-if** clause — the condition that puts a settled decision back on the
table. That stops re-litigation and flags when a parked call has gone live
again.

## Superseding a decision

When a decision is superseded, the old section stays with a one-line superseding
pointer, and every *mechanism* description elsewhere gets updated to the new
truth. A stale architecture section outlives its decision otherwise: gitlore's
branch model was rewritten by D17 in 2026-07-18, and the Architecture "Branch
Model" section, D2, D3 and four rejected-alternative rows still described named
per-worktree branches ten days later.

A past decision whose rationale looks thin on re-reading gets **re-examined, not
treated as settled**; new context or fresh eyes can invert it. In the 2026-05-19
handoff brainstorm the user pushed back on the "no SessionStart hook, use
`@`-ref instead" decision. Its arguments ("no hook overhead", "no race
conditions") were both negligible in practice, while the cost of the chosen path
(a whole user-facing setup skill, an entire silent-failure mode) was structural.
The inversion produced a cleaner architecture.

- When citing a past decision, check whether the *reasons* it gave still hold.
  Don't just relay the conclusion.
- If the user challenges a documented decision, default to honest re-examination
  over defending the doc.
- Be willing to write a new spec that inverts an old one. Spec history in git is
  the audit trail; the latest doc is the current answer.

The new dated entry leads with the motivating fact (the incident that forced the
change — load-bearing, must survive), then the positive design and its
reasoning. Cut the museum narrative: deletion inventories ("extract.py loses
functions A, B, C…"), "it no longer does what it used to" framing, and
inversion-bookkeeping ("this supersedes decision X and Y"). Old sections stay,
with a terse superseding *pointer* stating current truth — that one line is
where "what changed" bookkeeping goes, not the prose. "Retained as a record of
what was cut" is not a justification.

## Write-time record vs. current truth

Two rules pull in opposite directions and are resolved by *which artifact* is
being written:

- **A dated design-decision or changelog entry is a record.** It captured the
  thinking at the time, with the information available then. Its rationale
  paragraphs (why option A was chosen, what trade-offs were weighed) are the
  audit trail and are preserved, dates and all.
- **A memory file, a code comment, or a doc's standing description of how the
  system behaves is current truth.** Written present-tense, plainly, with no
  "used to be" framing. Leaving these stale doesn't preserve thinking, it
  actively misleads.

Inside a design doc the same split applies section by section: **rationale
paragraphs** — preserve; **mechanism descriptions** (how the system behaves now,
what happens in edge case X) — update. In the 2026-05-19 v0.3.0 release two
reviewers split on exactly this for handoff's `## Activation` chosen-pattern
bullet and `### Missing-extract edge case` (now in
`docs/changelog/2026-05-19-original-activation-and-loading.md`), both still
describing the abandoned `@`-ref load chain. The spec reviewer wanted them left
as history; the code-quality reviewer called them current-behavior descriptions.
The code-quality framing was correct — fixed in commit `7c0471b`.

## Present-tense standing truth

In memory bodies, code comments, and docs, state what is true now. Phrasings
like "no longer true", "X does now fire (contrary to before)", "not what it
was", "the earlier note said Y" read as museum-keeping: a fresh reader who never
held the old belief doesn't need the rebuttal, and the temporal framing dates
and rots. Git history already preserves what changed — the diff *is* the
changelog. David flagged this twice in one session (a memory body and a code
comment), explicitly extending it from memories to comments and docs.

The vantage is the future reader's, not today's. That reader holds none of
today's state: no pending action, no session about to end, no change halfway
applied. So a body that says "needs a fresh session", "still to be applied", or
"the new wording is going in" names nothing they can act on and is false as soon
as the state moves. Write the standing fact and let the transient half go to the
task file, the todo list or the commit message.

Sanctioned exception: inoculating against a *live* wrong belief still in
circulation — and even then one present-tense clause ("a widespread belief holds
the opposite; test before trusting it"), never a narration of the old version.
**A commit message is exempt:** it *is* a changelog, so before→after phrasing
belongs there.

Distinct but adjacent: don't claim a behaviour *changed* from a single
current-state test — a broken prior config or a different environment explains
it equally, so a delta isn't visible in one snapshot.

## Compact prose

"Less connective wordage. Same sense." (2026-07-12, handoff compaction
discussion.) Connective tissue — transitions, framing, restated context,
re-summaries of what was just said — costs reading time without adding content;
ddaa reads fast and challenges substance directly. State claims and their
grounds directly; drop transitional sentences and hedged framings.
**Compactness is cutting connectives, not dropping content** — completeness of
substance stays. Same instinct applied to hook output: [[hook-output-channels]].

Related: [[examine-evidence-drift]], [[loose-generation]].
