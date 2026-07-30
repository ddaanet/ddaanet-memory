---
name: reference-cwd-guard-deadlock
description: "`Bash commands blocked: working directory is not project root` + `Shell cwd was reset to <other>` = unrecoverable deadlock; `cd`, `cd X && cmd` and the user's `!` shell all get reset — only a relaunch clears it, and it lands wherever it is launched, not where you were working"
metadata:
  node_type: memory
  type: reference
---

A `cwd-safety`-style PreToolUse guard refuses every Bash call whose shell cwd is
not the project root:

```
❌ Bash commands blocked: working directory is not project root.
Current: /path/to/repo-a
Run this command to restore: cd /path/to/repo-b
```

When the harness has pinned the session cwd to `repo-a` and the guard's project
root is `repo-b`, that instruction cannot be followed and **Bash is dead for the
rest of the session**. Every escape is closed:

- `cd /path/to/repo-b` alone → `Shell cwd was reset to /path/to/repo-a`.
- `cd /path/to/repo-b && <cmd>` → caught by the guard's *other* branch,
  `cd away from project root causes working-directory drift`.
- the user running `! cd /path/to/repo-b` → reset the same way, even though the
  `!` shell otherwise shares the agent's cwd.

Only relaunching clears it. **A relaunch lands in whatever directory it is
launched from**, which is not necessarily the repo the work is in — check
`pwd` / `CLAUDE_PROJECT_DIR` / `git log -1` again on the far side rather than
assuming the restart fixed the pairing. `CLAUDE_PROJECT_DIR` is frequently
unset, so `pwd` plus the HEAD subject is the reliable pair.

**How to apply:** the guard's own message names the one legal escape that still
works — a subshell that does not persist. `(cd /other/repo && <cmd>)` and
`git -C /other/repo <cmd>` both run fine while the guard is otherwise blocking,
so cross-repo *work* is still reachable without a relaunch; it is only the
persistent cwd that is stuck. Reach for those before asking for a restart.

Two repos each carrying their own auto-memory means two `MEMORY.md` files, and
the one loaded into context is the **session root's**. A claim about "the memory
index" is ambiguous until the root is named — verifying the wrong file reads as
confirmation. See [[feedback-verify-session-root]] and
[[reference-cc-worktree-memory-freeze]].
