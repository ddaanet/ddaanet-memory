---
name: feedback_index_compaction_triggers
description: "compacting a memory index cuts symptoms and identifiers before you notice; count each distinctive token index-wide before and after, and treat any token falling to zero — or surviving on the wrong line — as a defect"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 40ca6ee7-236b-45e8-9428-2f7bbdd25844
  modified: 2026-07-26T14:40:47.906Z
---

An index line's only job is to let an agent decide read-or-skip. Recall usually
has to fire on a string met **mid-task** — a git error, a flag, a filename, a
symptom in a tool result. Shortening a line is therefore not a prose edit: if
the distinctive string goes, the agent hits the error and never matches the
line, and the memory is lost while the file still exists.

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

Related: [[feedback_verify_at_source]] (confirm a reported loss in the file
before acting), [[reference_cc_memory_retrieval_agentic]].
