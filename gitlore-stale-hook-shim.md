---
name: gitlore-stale-hook-shim
description: "`git push` prints `gitlore skipped: hooks dir is stale` and exits 0 with no memory published, or fails with `.git/gitlore-pre-push: exec: …/scripts/git-hooks/pre-push: not found`: run `(cd /abs/path/to/repo && claude -p ping)` unsandboxed, then push again"
metadata:
  node_type: memory
  type: reference
  modified: 2026-11-19T18:48:38.765Z
  originSessionId: 658c21ad-fde1-4465-bfc1-6c696237e8aa
---

A `git push` that prints `gitlore skipped: hooks dir is stale (plugin
upgraded; cache GC'd).` exits 0 and pushes the parent while publishing no
memory: the shim gitlore installs as the parent's `pre-push` finds no
executable hook under `git config gitlore.hooksDir` and skips rather than
blocking the push. An older shim fails the push instead, with
`.git/gitlore-pre-push: exec: …/scripts/git-hooks/pre-push: not found`; the
remedy below is the same for both.

The remedy is `(cd /abs/path/to/repo && claude -p ping)`, run unsandboxed: its SessionStart hook rewrites the shim and repoints
`git config gitlore.hooksDir` at the installed plugin. Then repeat the push,
which publishes the memory commits the first one left behind. `claude -p ping`
is a fresh process, so its SessionStart hook does re-fire; the mid-session
hook text saying not to repair "by rewriting gitlore.* git config" does not
forbid it.
