---
name: git-subtree-ensure-clean-unscoped
description: "git subtree add/pull/push refuse on ANY working-tree modification via an internal, unscoped, unbypassable check — a pathspec fix at the call site can't work around it"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 088576b6-2351-4a78-b826-5679e810c7ce
  modified: 2026-08-11T16:05:14.705Z
---

`git subtree`'s `ensure_clean()` (in `/usr/lib/git-core/git-subtree`) runs
`git diff-index HEAD --exit-code --quiet` with no pathspec and dies "fatal:
working tree has modifications.  Cannot add." on any hit. There is no CLI
flag to scope or bypass it — not `--force`, not a pathspec argument to
`subtree add`/`pull`/`push` themselves.

This matters whenever some part of a repo is expected to be "dirty" by
design between commits — e.g. a gitlore-mounted memory submodule sitting at
a gitlink SHA ahead of what HEAD records. A caller's own pre-check can be
scoped with a pathspec exclusion (`git diff --quiet HEAD -- . ':(exclude)some/path'`)
to tolerate that, but `git subtree` will still refuse on its own internal
check regardless — the caller's fix just trades a clear, on-brand error for
subtree's more cryptic one, without letting the operation actually succeed.

**How to apply:** before assuming a pathspec exclusion added at a call site
will let `git subtree add`/`pull`/`push` proceed past an expected-dirty
path, check whether the actual failure is coming from `ensure_clean()`
inside `git-subtree` itself (grep `/usr/lib/git-core/git-subtree` or the
platform equivalent). If so, the only ways through are: make the working
tree genuinely clean first (a real commit), or avoid `git subtree` there.
