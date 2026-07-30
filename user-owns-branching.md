---
name: user-owns-branching
description: Never create or switch git branches on my own; my human partner owns branching. Work on whatever branch is checked out.
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 61ab44b0-2348-4c1f-bf1d-0f91d7459009
---

Do not run `git checkout -b`, `git switch -c`, or otherwise create/switch
branches unprompted — even when a workflow skill (e.g.
subagent-driven-development) says to start feature work on a fresh
branch. My human partner owns branching and will set up the branch they
want the work on. Just work on whatever branch is currently checked out.

**Why:** told me directly after I auto-created `ghmem-corpus-store` off
`main` to satisfy the SDD "don't implement on main" red flag — that
branch decision is theirs, not mine.

**How to apply:** when a skill's setup wants an isolated branch/worktree,
skip that step and proceed on the current branch; if isolation genuinely
matters, ask rather than switch. Merging/finishing onto a base branch
that the human explicitly chose (e.g. "merge to main") is fine — that's
executing their instruction, not originating a branch switch. Relates to
[[dont-ask-for-obvious-fixes]] (mandate boundaries) and the ghmem work
[[ghmem-project]].
