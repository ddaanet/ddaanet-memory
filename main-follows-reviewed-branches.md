---
name: main-follows-reviewed-branches
description: Work happens on dev or a feature branch; main only merges or fast-forwards after that branch has been through review — never ask to move main
metadata:
  node_type: memory
  type: feedback
  originSessionId: 0c76dad6-3994-43d8-8feb-ca62743586f4
  modified: 2026-10-13T12:18:20.650Z
---

Work happens in branches (`dev`, or a feature-specific branch). `main` merges
or fast-forwards only after that branch has been through review — including
but not limited to `/code-review`, `/codex:review` and `/simplify`. Do not ask
my human partner whether to move `main`, and do not carry "move `main` to
`dev`" as a pending item waiting on their say-so.

**Why:** stated by my human partner on 2026-10-13 in sandbox-lies, as a policy
change made for compatibility with every git code review tool: those tools
review a branch against its base, so work committed straight to `main` leaves
nothing to diff. They said "stop asking for main" after a handoff listed
moving `main` as waiting on them.

**How to apply:** commit on whatever branch is checked out
([[my-human-partner-owns-branching]] still holds: never create or switch one
unprompted). In a handoff or todo, record what review a branch has and has not
had, not a request to promote it. Applying this beyond sandbox-lies is my
reading of "my policy"; the statement named no repository.
