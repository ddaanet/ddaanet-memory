---
name: index-compaction-triggers
description: "classify WHEN / HOW / acted-inline before touching a line, and relocate the acted-inline class to CLAUDE.md or shared-claude.md rather than shortening to hit a byte target; gitlore's advisory 25600-byte budget vs CC's 24.4KB loader cap, the gate that silently drops the tail; over budget is NEVER a reason to under-trigger a new/updated fact — pressure means retire, not ship an unroutable line; audit plain words, not just backticked literals"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 40ca6ee7-236b-45e8-9428-2f7bbdd25844
  modified: 2026-08-10T18:55:50.751Z
---

An index line's only job is to let an agent decide read-or-skip. Recall usually
has to fire on a string met **mid-task**: a git error, a flag, a filename, a
symptom in a tool result. Shortening a line is therefore not a prose edit.
Shortening a line to hit a byte target strips exactly the distinctive tokens —
specific commands, flags, error-message substrings, file paths, version
numbers — that routing depends on. If the distinctive string goes, the agent
hits the error and never matches the line, and the memory is lost while the
file still exists.

The index is not the place to put something because it should always be in
context. A fact that must be present without being looked up belongs in
`CLAUDE.md`, `~/.claude/CLAUDE.md`, or a shared tier's `shared-claude.md` when it
holds across every repo — those load whole and are meant for it. Loading the index
every session is a consequence of it being a routing table, not a second purpose to
write for.

**Every line is one of three kinds, and the kind tells you what its trigger
surface is.** Classify before touching a line:

- **WHEN** — fires on a symptom met mid-task: an error string, a flag, a config
  key, an observed misbehaviour. The symptoms and identifiers *are* the trigger;
  the prescription is payload.
- **HOW** — fires on a task or decision you are starting. The **condition** is the
  trigger; the rule content is payload. The condition often hides inside what
  reads like prescription: "ask what vocabulary the rule needs, not which layer
  catches it first" is the trigger ("which layer should validate this?"), not the
  rule.
- **Acted-inline** — there is no lookup step. Model tier while typing `model:`
  into an `Agent` call, how to answer a correction, what belongs in always-on
  context. For these the index *is* the point of use, so a hook that routes but
  does not say which side the memory lands on is worthless. **These do not belong
  in the index at all** — they belong in the always-loaded file, and moving them
  there is the one compaction lever that works (below).

A line that resists compression is usually already pure routing. A line that
compresses easily is usually acted-inline and was never index material.

**Why the pressure exists at all:** the binding constraint is bytes, not
lines — the index blob loads verbatim into every session. Measured
2026-07-22 on this store: one project-state line alone was 3.3KB (18% of
the whole index) and the top five were 31%, while every behavioral
directive line together was a small fraction. Curation effort belongs on
the long state/reference lines, not on trimming everything uniformly.

**Two budgets, and only one of them is a gate.** gitlore's own check
(`GITLORE_INDEX_BUDGET_BYTES`, 25600, warning at 80%) reports and never refuses
composition — it stayed quiet-but-unheeded at 103%. **Claude Code's own loader
is a hard cap:** past roughly 25 KB it silently truncates the always-loaded
index and says so once (`MEMORY.md is 25.8KB (limit: 24.4KB) — Only part of it
was loaded`). Observed 2026-07-24 at 26754 B: the last six entries never reached
the session. Check whether a size constraint is a hard gate or merely advisory
before treating it as a forcing function for content loss.

**Truncation takes the tail, and composition puts project lines there.** The
composed root index is `[tier blocks, manifest order] → [project bare-path
lines]`, so an overflowing index loses the *project-local* memories first — the
ones most specific to the repo you are in. That ordering is correct on its own
terms (broad facts first, specific ones override), which is exactly why the cap
must be treated as a failure mode to stay under rather than a degraded mode to
tolerate.

**The failure has a direction.** Trimming preserves PRESCRIPTIONS and cuts
SYMPTOMS, CONTRASTS and IDENTIFIERS, because that is what reads well as prose.
It is exactly backwards for a routing table: an agent arrives holding a symptom
and needs to find the prescription, not the reverse.

**Count tokens, don't eyeball presence.** For every distinctive literal, compare
occurrences index-wide before and after. Two failure grades:

- a token falling to **zero** — the memory is unreachable by that route;
- a token falling to **one, on a different line** — worse than deletion,
  because the surviving occurrence now MISROUTES. Dropping
  `Unable to create '.git/index.lock': File exists` from the hook-env-leak entry
  left one `index.lock` in the whole index, on the sandbox entry, so an agent
  hitting that error in a hook would be routed to a memory that blames the
  sandbox — the wrong diagnosis.

**What is safe to cut**: rationale, dates, provenance ("verified 2026-…"),
cross-references, explanatory clauses. **What is not**: quoted error strings,
config keys, env vars, flag and event names, file paths, the observable symptom,
and any contrast whose value was the disambiguation (`fetch -q` vs `push -q`).
A hook naming a count rather than a thing ("six-section doc") is unmatchable.

**The levers that buy real headroom are relocating the acted-inline class,
retiring reconstructable facts, and merging overlapping ones** — never stripping
trigger tokens from surviving lines. Relocation is the largest and is treated in
full below. Merging buys less than it looks: index cost is trigger surface, and a
merged line must still carry both entries' distinctive literals or it
misroutes, so only the shared framing prose collapses. Measured: three merges
collapsing 103 entries to 100 on a 23.8KB index netted 39 bytes — 0.16% — and
one of the three left the index *larger* than it found it, because the
surviving line had to carry both trigger sets plus the connective prose joining
them. Merge for routing (two entries competing on one trigger hand an agent
half a rule), never for headroom. Retiring a fact is what actually removes
bytes, because its triggers go with it. A topic file left
intact is no defence — an index line that no longer carries what a recall pass
would match on is unreachable however complete the file behind it is.

**Being over budget is never a reason to under-trigger a new or updated fact.**
The pressure argues for retiring facts, not for shipping a line that cannot
route to the one just written. A fact whose index line omits the literal a
recall pass would match on is already lost, so trading its triggers for bytes
buys headroom by destroying exactly what the headroom is for — and it does it
silently, since the file is there and looks complete. Write the triggers the
fact needs; if the index is over the cap, that is the retirement decision
coming due, and it is a separate act with its own judgement. Observed
2026-08-01: an addition to the hook-schema line was self-reverted on the
reasoning that 100 bytes would push 100 bytes off the invisible tail. Correct
about the arithmetic, wrong about which side pays — the reverted line was the
only route to a fact written that same session.

**Relocating the acted-inline class is the lever that actually works.** Moving a
standing rule out of the index and into `CLAUDE.md` or `ddaanet/shared-claude.md`
is right in kind — that is where a rule that must act without being looked up
belongs. It buys no *total* context headroom, because the destination loads whole
every session too, and a bullet written from scratch can exceed the line it
replaced (measured once at 14 relocations: −2,200 B of index, +2,692 B of
`shared-claude.md`).

**But total context is not the constraint that bites — the index cap is.** Only
the index is silently truncated. My human partner settled this explicitly:
"don't sweat the shared-claude.md tokens, it's a different limit, and the goal
here is fixing memory index overflow." Measured on the 2026-08-01 pass: 24
acted-inline lines out of the index cut 3,491 B from the capped file and grew
`shared-claude.md` from 5,765 to 14,070 B — and that 3.5 KB is what paid for
restoring every trigger and carve-out a preceding trim pass had destroyed. The
index landed at 22,168 B with 2.8 KB of headroom and zero misroutes.

Relocation is not a byte trick. It is the correct home for a rule with no lookup
step, and the byte relief is a consequence. Carry each rule across **whole, with
its carve-outs** — read the topic file, not just the index line, because the
"how to apply" section is where the exceptions live.

**A compressed rule must keep its exceptions.** Squeezing a fact into an
always-on bullet drops the "how to apply" section, and carve-outs live in there:
the scope limit, the permitted case, the "does not apply to X". A rule that lost
its exception is not shorter, it is *wrong* — and wrong in always-on context,
where nothing routes an agent to the file that would correct it. Six of those 14
relocations dropped something that was not procedure; the GitHub-body rule kept
"every newline renders as `<br>`" and lost "blank lines between paragraphs and
list items are still correct", which inverts it.

**Retiring a fact inverts this audit, and "that workstream is finished" is a
premise to verify.** For a retirement the literals are *supposed* to fall to
zero, so the question becomes whether one falls to a single occurrence on an
unrelated line — `400` surviving only on "400-line PR cap is soft" sends an agent
asking about the hard per-file gate to the opposite answer. And a cluster retired
as a closed workstream needs the closure checked against what still ships: six
tmux facts went on the assumption that probing was done, while two skills built
on them were live in the session's own skill list. Restoring five wrongly-retired
facts cost 1.3 KB against a 7 KB overshoot — the budget was never what made those
retirements wrong.

**Have someone else audit the diff.** Self-review does not catch this — the
author knows what each line means and so reads the trimmed version as adequate.
An adversarial pass briefed to falsify "I cut only elaboration" found 15 bad
lines in an already self-checked diff, and its own verification pass then found
six more it had missed.

**Run two auditors on different briefs; they find disjoint classes.** On the
2026-08-01 pass one was briefed on trigger loss (tokens to zero, tokens fallen to
one on the wrong line, conditions cut in place of prescriptions, collapsed
disambiguations, hooks naming a count) and one on meaning damage (rules that lost
an exception, inversions by truncation, neutral "X vs Y" hiding which side the
memory lands on, scope drift). They returned 33 and 25 findings with little
overlap. Brief each to falsify, not to confirm, and tell each to re-verify its own
findings after declaring itself done.

**A backtick-and-quote sweep is not sufficient as a literal audit.** My own
self-audit extracted only backticked and double-quoted spans and reported *zero*
misroutes; the auditor's plain-word lost-vocabulary sweep — every word present in
BEFORE and absent in AFTER, 464 of them — found four real misroutes plus
`heredoc`, `precompact`, `mtime`, `unsandboxed`, `parallel` and every model name
sitting at zero. Bare identifiers carry routing weight without ever being marked
up. Sweep plain words too.

**Trimming a whole index is measured as the wrong lever.** Cutting prescriptions
while keeping symptoms — the direction this file endorses — took a 32,405 B index
to 24,316 B (−25%). Two adversarial audits then found 58 findings: 4 misroutes,
17 tokens at zero, 6 rules that lost their exception, and one line rewritten into
a claim that was simply *false* (a hook's `additionalContext` "size limit", when
the fact is that a large payload is spilled to a file with ~2 KB inlined). About a
fifth of touched lines took real damage, and repairing them consumed almost the
entire 8 KB the trim had bought. Two things make the compressible surface far
smaller than it looks: prescriptions carry trigger literals (`precompact`,
`Haiku`, `CLAUDE.md`, `heredoc`, `fsync`), and carve-outs live in the prescription
half, so a "shortened" rule is frequently a wrong one.

What *did* survive as a real gain: hooks that merely restated their own title
(`choosing a hook output channel:` on a line titled *hook output channels*) and
boilerplate like "full state and history in the file", which every entry implies.
Delete those freely — they cost bytes and route nothing.

**A silent trim buried in an unrelated commit is not a divergence to weigh —
just restore it.** Two lost triggers (`phantom dotfiles`, `own hooks.json
unlink EROFS`) had their removal unmentioned in the commit message that carried
it (that commit documented dropping two different, unrelated facts). Framing
this as "should I fork away from upstream" is the wrong question when it's all
one shared history with no actual conflict — restoring a token nobody
deliberately removed is just the next commit, not a fork to justify.

**Adversarial audit, confirmed working.** After a real compaction pass
(23,874→22,193 bytes), a fresh non-fork agent independently re-derived the
literal list from `git diff` rather than trusting the compacting agent's
self-report, and confirmed 168/174 distinctive tokens survived with zero
misrouted and the root/tier mirror still 1:1. Cheap enough to run every time —
do not skip it because the self-report already claims a clean audit.

**Compaction's real diff is wider than the index file.** Editing `MEMORY.md`
triggers `index-compose`/`index-sync` to rewrite the `description:` frontmatter
of every touched fact file to match — one 2-line index edit produced a 57-file
diff (2 index files + 55 fact-file frontmatter updates). Review the whole
submodule diff before committing a compaction, not just the index files.

Related: [[memory-writing]] (what belongs in the fact file the
line points at); background on why the index is the only reliably-recalled
surface in the gitlore repo's `docs/references/cc-memory-retrieval.md`, whose
changelog also holds the recall experiments (`2026-07-22-active-recall-fr16-d18`,
`2026-08-08-recall-skill-no-hook`) — check there before re-running one.
