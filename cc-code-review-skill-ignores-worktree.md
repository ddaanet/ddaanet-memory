---
name: cc-code-review-skill-ignores-worktree
description: "a subagent told to run `/code-review` on a worktree's diff reviews the session's main checkout instead — the skill reads the session cwd, not the directory the prompt names; have the worktree agent commit on its branch and review that sha"
metadata:
  node_type: memory
  type: reference
  originSessionId: 69962a8e-ad4b-459a-9fef-75e44b55be27
  modified: 2026-11-19T11:04:00.964Z
---

The `code-review` skill, invoked by a subagent through the Skill tool with a
worktree path in its brief, reviews the uncommitted diff of the session's main
checkout, not the worktree's: the reviewer reports that the skill's fork "ran
in the main checkout and reviewed that checkout's own uncommitted diff". A
`cd` in the agent's Bash calls does not reach the skill's own fork. Given a
commit (`code-review medium <sha>`), the skill reviews that commit, wherever
its branch lives.

**How to apply:** a worktree agent commits its work on the worktree's own
branch (the commit-msg hook wants a conventional prefix), the reviewer runs
`code-review medium <sha>` on that commit, and the main thread applies the
worktree's diff to `dev` after the gate; the worktree branch is never merged.
A hand review of `git -C <worktree> diff` is a fallback, not the loop's
review. Candidate for `edify:review`, which owns the per-commit loop.
Related: [[cc-subagent-worktree-base]].

Seen on CC 2.1.328.
