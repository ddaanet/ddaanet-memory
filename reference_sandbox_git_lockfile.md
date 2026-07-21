---
name: reference_sandbox_git_lockfile
description: "sandboxed `git add` can leave `.git/index.lock` behind → the NEXT commit's pre-commit hook dies on \"File exists\"; re-run unsandboxed"
metadata: 
  node_type: memory
  type: reference
  originSessionId: e2a4dfcf-7b44-4f1c-bfa8-47c72f899330
  modified: 2026-07-21T19:43:37.270Z
---

A `git add` run inside CC's Bash sandbox can succeed at staging while failing to
remove `.git/index.lock`. Nothing looks wrong until the *next* command: `git
commit` runs its `pre-commit` hook, the hook's own `git add` tries to take the
lock, and git reports `Unable to create '.git/index.lock': File exists. Another
git process seems to be running` — naming a concurrent process that does not
exist. Re-running the identical command with `dangerouslyDisableSandbox: true`
succeeds.

Observed 2026-07-21 in gitlore: `git add <file> && git commit` in one sandboxed
Bash call. The staging succeeded (`git status` showed `A `), the commit aborted,
and git cleaned the lock on abort — so by the time you look, the lock is gone
and the evidence is destroyed.

Diagnostic that settles it: `GIT_TRACE=1 git commit` unsandboxed. It prints
`run_command: GIT_EDITOR=: GIT_INDEX_FILE=.git/index .../hooks/pre-commit` —
proving git has NOT taken `index.lock` when `pre-commit` runs, so a hook's bare
`git add` is legitimate and a lock present at that moment is a leftover, not
contention.

Do not "fix" the hook in response. A staging hook's `GIT_INDEX_FILE`
capture/restore ([[reference_git_hook_env_leak]]) is a different bug with the
same error string: that one fires under `-a`/pathspec commits where git really
does hold the lock. Distinguish by whether an unsandboxed retry succeeds — a
real hook bug fails either way.

Related: [[feedback_strict_sandbox_git]], [[feedback_git_status_sandbox]].
