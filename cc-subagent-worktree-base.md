---
name: cc-subagent-worktree-base
description: "a subagent `isolation: worktree` shows old files at the `main` commit because unpushed `dev` was not its base: `worktreeBaseRef` / `worktree.baseRef` = `head` branches from local HEAD; the auto-mode classifier denies `git merge --ff-only` and `git reset --hard` run by an agent inside a worktree"
metadata:
  type: reference
---

A subagent launched with `isolation: worktree` branches from the remote's
default branch, not from the checked-out one. A local branch that is not
pushed therefore shows up in the worktree as the old files at the default
branch's commit, and the agent's work lands on the wrong base.

- **Fix:** set `worktree.baseRef` to `head` (`/config`, key `worktreeBaseRef`);
  the worktree then branches from the local HEAD. The setting takes `fresh` or
  `head`; a branch name is not accepted.
- **Check on first use:** run `git log -1` in the worktree and compare it with
  the commit the brief names, and have the brief tell the agent to stop on a
  mismatch.
- **Merge from the main tree.** The auto-mode classifier denies
  `git merge --ff-only` and `git reset --hard` when an agent runs them inside
  a worktree, so the agent commits on its own branch and the parent session
  merges it.

Seen on CC 2.1.28x. Related: [[cc-worktree-bash-guard]],
[[cc-worktree-memory-freeze]].
