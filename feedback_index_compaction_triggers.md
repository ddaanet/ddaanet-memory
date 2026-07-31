---
name: feedback_index_compaction_triggers
description: "an index line's ONLY job is read-or-skip routing — want it always in context, use CLAUDE.md/~/.claude/CLAUDE.md; never shorten one to hit a byte target — trimming keeps prescriptions and cuts symptoms/identifiers; gitlore's 25600-byte budget and its 17.1KB compaction prompt are advisory, CC's 24.4KB loader cap is the gate and silently truncates the tail (project lines first); count each literal index-wide before and after — falling to zero is unreachable, falling to one on another line MISROUTES; have someone else audit the diff"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 40ca6ee7-236b-45e8-9428-2f7bbdd25844
  modified: 2026-07-31T09:34:14.504Z
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
`CLAUDE.md`, or `~/.claude/CLAUDE.md` when it holds across every repo — those
load whole and are meant for it. Loading the index every session is a
consequence of it being a routing table, not a second purpose to write for.

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

**The only levers that buy real headroom are merging overlapping facts and
retiring reconstructable ones** — never stripping trigger tokens from surviving
lines. Merging buys less than it looks: index cost is trigger surface, and a
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

**Have someone else audit the diff.** Self-review does not catch this — the
author knows what each line means and so reads the trimmed version as adequate.
An adversarial pass briefed to falsify "I cut only elaboration" found 15 bad
lines in an already self-checked diff, and its own verification pass then found
six more it had missed.

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

Related: [[feedback_verify_at_source]] (confirm a reported loss in the file
before acting); [[feedback_memory_writing]] (what belongs in the fact file the
line points at); background on why the index is the only reliably-recalled
surface in `docs/references/cc-memory-retrieval.md`.
