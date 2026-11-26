---
name: release-resume-verify-step
description: "a frame saying \"resume `/edify:release` at step N\" is a status claim: before skipping steps 2–5, check in `git log main..dev` that the simplify commit and the release-review fix rounds come after the last content commit, else restart at step 2"
metadata:
  node_type: memory
  type: feedback
  originSessionId: de68b711-62d0-4436-b59a-bcb7cc5bee4e
  modified: 2026-11-26T11:35:08.602Z
---

A task frame whose task text opens "resume `/edify:release` at step N"
states where the release stood when it was written; content commits can land
on `dev` after it, or the frame can conflate per-commit medium reviews
(`edify:review`) with the release's own steps 2–5 (simplify, the Fable high and
Codex astra reviews over `main...dev`, triage, the medium loop). Skipping to
step 6 on the frame's word alone ships a delta no release review covered.

Before resuming past step 2, read `git log --oneline main..dev` and confirm
that the simplify commit (`refactor:`) and the release-review fix rounds sit
after the last content commit. If any content commit follows them, restart at
step 2 over `main...dev`. Whichever step the resume starts at, the reply says
which earlier steps are taken as done and on what evidence.

**Why:** a release resumed at step 6 had a simplify pass and one review-fix
round early in its delta, followed by about fourteen more content commits that
had only per-commit medium reviews; my human partner caught the skip at
preflight: "release should mention simplify and code reviews, you behave as if
you skipped this step entirely."

**How to apply:** at `edify:release` step 1, whenever a frame names a step past
2. Interim home: when `edify:release` step 1 carries this check, delete this
file. Related: [[release-review-slicing]].
