---
name: feedback-reconstructable-two-categories
description: "by-harness (deterministic, free → drop it) vs by-model-inference from a paraphrase (lossy, fails silently → carry it); name the reconstructor before dropping anything"
metadata:
  node_type: memory
  type: feedback
---

A residual analysis that drops everything "reconstructable" is applying one
label to two different things.

**Reconstructable by the harness** — `git status`, `git log`, the files-touched
set, the working tree. Recomputed deterministically at read time, free, correct
by construction, and *fresher* than any snapshot could be. Carrying it is
duplication that goes stale; drop it and defer.

**Reconstructable only by model inference** — the state of an in-flight task
list after a compaction, for instance. The successor has a paraphrase and has to
work out what is still open. That inference is lossy, and its failure is
**silent**: work already finished gets done again, with nothing signalling that
anything went wrong.

**Why:** the two share a word and nothing else. The test that separates them is
*who* does the reconstructing and *what happens when it is wrong* — a harness
recomputation cannot be wrong, a model inference from a summary routinely is,
and an avoidable inference whose failure is invisible is worth paying a file
for.

**How to apply:** before dropping something as reconstructable, name the
reconstructor. If it is a script or the harness, drop it. If it is the model
working from compacted context, carry it — and carry only the part that
inference gets wrong. For a task list that is the *open* half; the finished half
really is in `git log`, and keeping it around is what makes a stale ledger
dangerous. Applied in [[project_handoff_todo_ledger]]; the same distinction is
why the task frame defers to `gitStatus` but not to the summariser.
