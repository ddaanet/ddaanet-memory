---
name: feedback-design-doc-writing
description: "dated design entries are write-time records (keep rationale); memory/docs/comments state present-tense truth; six-section doc: FR, NFR, architecture, decisions, rejected alternatives, changelog — the changelog its own file, the design present-tense what/how/why; `(implemented <date>)` tags and status blobs are changelog material, prose subsections beat a big table"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 618e80ab-c1a4-41c0-94c9-07e03fa4af68
  modified: 2026-07-29T14:51:19.189Z
---

A design doc is a living doc, not a one-off spec. Six sections:
functional requirements, non-functional requirements, architecture,
design decisions (with rationale), rejected alternatives, changelog —
and the changelog is a **separate file** the doc points at
(`docs/changelog.md` in gitlore) once it outgrows a short list. Past a
certain size that file is only the index and each change gets its own dated
file beside it — `docs/changelog/<date>-<title>.md` in handoff, one line per
entry in the index.
Plans and specs go in a root `plans/`; the spec/design doc is a **single living
doc for the project** — `docs/design.md`. The rule is one living doc, not one
filename; check where the project actually keeps it before creating a second.

**Why:** it captures the *why* alongside the *what*, so future sessions
don't re-litigate settled decisions. The rejected-alternatives section
matters most — it prevents revisiting approaches already ruled out for
good reason.

**How to apply:** when updating, add to the changelog and update the
relevant section. Don't flatten or summarize decisions — preserve the
rationale.

## Two documents, two jobs

The living design answers **what, how and why**, in the present tense: what
the system does, how it is built, why it is built that way. The changelog
answers **how we got here** — one dated entry per change, each a write-time
record keeping the reasoning available at the time.

That split decides where text goes when a doc is cleaned up. Implementation
status ("slice 3-ii done 2026-07-21, 37 cases in …"), `(implemented
2026-07-15)` tags stamped on a mechanism paragraph, and "this retired the
two earlier point-fixes" narratives are all changelog material: in the
design they are noise, and they rot. Dates that *ground a claim* stay
wherever the claim is — "verified against CC 2.1.150", "measured over 76
bullets", "eval over 528 transcripts" — because deleting them turns
evidence into assertion.

Prose beats a big table for anything with a paragraph's worth of reasoning
in a cell: a rejected-alternatives or decisions table hides the rationale in
one long cell and forces every entry into the same width. Subsections with a
bolded lead-in read the same at a glance and let the argument breathe.
Keep tables for genuinely tabular data — short, uniform, scannable cells.

When a decision is superseded, the old section stays with a one-line
superseding pointer, and every *mechanism* description elsewhere gets
updated to the new truth. A stale architecture section outlives its
decision otherwise: gitlore's branch model was rewritten by D17 in
2026-07-18, and the Architecture "Branch Model" section, D2, D3 and four
rejected-alternative rows still described named per-worktree branches ten
days later.

## Write-time record vs. current truth

Two rules pull in opposite directions and are resolved by *which
artifact* is being written:

- **A dated design-decision or changelog entry is a record.** It
  captured the thinking at the time, with the information available
  then. Its rationale paragraphs (why option A was chosen, what
  trade-offs were weighed) are the audit trail and are preserved, dates
  and all.
- **A memory file, a code comment, or a doc's standing description of
  how the system behaves is current truth.** Written present-tense,
  plainly, with no "used to be" framing. Leaving these stale doesn't
  preserve thinking, it actively misleads.

Inside a design doc the same split applies section by section:
**rationale paragraphs** — preserve; **mechanism descriptions** (how the
system behaves now, what happens in edge case X) — update. In the
2026-05-19 v0.3.0 release two reviewers split on exactly this for
handoff's `## Activation` chosen-pattern bullet and
`### Missing-extract edge case` (now in
`docs/changelog/2026-05-19-original-activation-and-loading.md`), both still
describing the abandoned
`@`-ref load chain. The spec reviewer wanted them left as history; the
code-quality reviewer called them current-behavior descriptions. The
code-quality framing was correct — fixed in commit `7c0471b`.

## Present-tense standing truth

In memory bodies, code comments, and docs, state what is true now.
Phrasings like "no longer true", "X does now fire (contrary to before)",
"not what it was", "the earlier note said Y" read as museum-keeping: a
fresh reader who never held the old belief doesn't need the rebuttal,
and the temporal framing dates and rots. Git history already preserves
what changed — the diff *is* the changelog. David flagged this twice in
one session (a memory body and a code comment), explicitly extending it
from memories to comments and docs.

Sanctioned exception: inoculating against a *live* wrong belief still in
circulation — and even then one present-tense clause ("a widespread
belief holds the opposite; test before trusting it"), never a narration
of the old version. **A commit message is exempt:** it *is* a changelog,
so before→after phrasing belongs there.

Distinct but adjacent: don't claim a behaviour *changed* from a single
current-state test — a broken prior config or a different environment
explains it equally, so a delta isn't visible in one snapshot.

## Superseding a decision without museum narrative

A past decision whose rationale looks thin on re-reading gets
**re-examined, not treated as settled**; new context or fresh eyes can
invert it. In the 2026-05-19 handoff brainstorm the user pushed back on
the "no SessionStart hook, use `@`-ref instead" decision recorded in
handoff's `## Loading` section (now in
`docs/changelog/2026-05-19-original-activation-and-loading.md`). Its
arguments ("no hook overhead", "no race
conditions") were both negligible in practice, while the cost of the
chosen path (a whole user-facing setup skill, an entire silent-failure
mode) was structural. The inversion produced a cleaner architecture.

- When citing a past decision, check whether the *reasons* it gave still
  hold. Don't just relay the conclusion.
- If the user challenges a documented decision, default to honest
  re-examination over defending the doc.
- Be willing to write a new spec that inverts an old one. Spec history
  in git is the audit trail; the latest doc is the current answer.

The new dated section leads with the motivating fact (the incident that
forced the change — load-bearing, must survive), then the positive
design and its reasoning. Cut the museum narrative: deletion inventories
("extract.py loses functions A, B, C…"), "it no longer does what it used
to" framing, and inversion-bookkeeping ("this supersedes decision X and
Y"). Old sections stay, with a terse superseding *pointer* stating
current truth — that one line is where "what changed" bookkeeping goes,
not the prose. "Retained as a record of what was cut" is not a
justification.

## Compact prose

"Less connective wordage. Same sense." (2026-07-12, handoff compaction
discussion.) Connective tissue — transitions, framing, restated context,
re-summaries of what was just said — costs reading time without adding
content; ddaa reads fast and challenges substance directly. State claims
and their grounds directly; drop transitional sentences and hedged
framings. **Compactness is cutting connectives, not dropping content** —
completeness of substance stays. Same instinct applied to hook output:
[[reference-hook-output-channels]].

Related: [[harness-over-agent]] (a design principle whose *application*
can shift even though the principle holds),
[[feedback-examine-evidence-drift]], [[feedback-verify-at-source]],
[[feedback-loose-generation]].
