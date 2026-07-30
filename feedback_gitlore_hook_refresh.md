---
name: gitlore-hook-refresh
description: "Fix a stale gitlore pre-push hook by running `claude -p ping` in the repo to trigger the SessionStart hook"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 32f44f4e-e634-4e25-b276-e948b95f86ef
  modified: 2026-07-29T16:08:23.324Z
---

A `git push` that fails with
`.git/gitlore-pre-push: exec: .../gitlore/<version>/scripts/git-hooks/pre-push: not found`
has a stale hook path, not a broken push. Running `claude -p ping` inside that
repo — `(cd /path/to/repo && claude -p ping)` — repairs it: gitlore's
SessionStart hook rewrites `.git/gitlore-pre-push` to a shim that skips
gracefully instead of erroring, and repoints `git config gitlore.hooksDir` at
the installed version. Plain `git push` works afterwards.

**Why:** the hook records a version-specific script path, so any gitlore
upgrade or cache GC leaves it pointing at a directory that no longer exists,
and every push in that repo fails hard until something rewrites it. The
SessionStart hook is the only thing that does.

**How to apply:** reach for the ping, not `git push --no-verify` — bypassing
the hook skips gitlore's memory sync entirely and is a last-resort stopgap.
One repo hitting this usually means all of them will, since they share the
plugin cache. The workflow that produces these consumer pushes is
[[subtree-no-hand-edit]].
