---
name: feedback_index_compaction_triggers
description: "compacting a memory index cuts symptoms and identifiers before you notice; count each distinctive token index-wide before and after, and treat any token falling to zero — or surviving on the wrong line — as a defect"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 40ca6ee7-236b-45e8-9428-2f7bbdd25844
  modified: 2026-07-29T16:17:57.650Z
---

An index line's only job is to let an agent decide read-or-skip. Recall usually
has to fire on a string met **mid-task** — a git error, a flag, a filename, a
symptom in a tool result. Shortening a line is therefore not a prose edit: if
the distinctive string goes, the agent hits the error and never matches the
line, and the memory is lost while the file still exists.

**Why the pressure exists at all:** the binding constraint is bytes, not
lines — the index blob loads verbatim into every session. Measured
2026-07-22 on this store: one project-state line alone was 3.3KB (18% of
the whole index) and the top five were 31%, while every behavioral
directive line together was a small fraction. Curation effort belongs on
the long state/reference lines, not on trimming everything uniformly.

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

**Have someone else audit the diff.** Self-review does not catch this — the
author knows what each line means and so reads the trimmed version as adequate.
An adversarial pass briefed to falsify "I cut only elaboration" found 15 bad
lines in a diff I had already checked, and its own verification pass then found
six more it had missed.

**Resist an advisory byte target.** A hard truncation limit is a real failure
(the tail is dropped at load, silently). A softer "target" below it is not worth
trigger surface — see [[feedback_memory_index_lines_functional]], which warns
about exactly this pressure and is the rule the warning itself pushed me to
break.

**A silent trim buried in an unrelated commit is not a divergence to weigh —
just restore it.** Found two lost triggers (`phantom dotfiles`, `own hooks.json
unlink EROFS`) whose removal wasn't mentioned in the commit message that carried
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
before acting); background on why the index is the only reliably-recalled
surface in `docs/references/cc-memory-retrieval.md`.
