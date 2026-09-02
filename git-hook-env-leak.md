---
name: git-hook-env-leak
description: "a hook that stages: save/restore `GIT_INDEX_FILE` across `unset $(git rev-parse --local-env-vars)` or the `git add` dies on `Unable to create '.git/index.lock': File exists`; `index file open failed: Not a directory` is the leak itself; a standalone hook run exercises neither"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 39084536-e084-480e-981f-18bb2e894d63
  modified: 2026-07-21T07:48:50.678Z
---

`fatal: .git/index: index file open failed: Not a directory` raised inside a git
hook is leaked repo-local `GIT_*` reaching a `git -C <submodule>` call: the
inherited relative `GIT_INDEX_FILE` resolves under the submodule's cwd, where
`.git` is a gitfile. A sandbox can raise the same string for an unrelated
reason, so confirm the environment is actually leaking before treating it as
this bug.

A hook that clears that environment and **stages something itself** has to keep
one variable across the unset. `GIT_INDEX_FILE` names the index the in-flight
commit will actually read, so a `git add` run without it either targets the
wrong index or — under the two lock-file modes — dies on
`fatal: Unable to create '.git/index.lock': File exists`, which under errexit
blocks the commit.

Capture it before the blanket unset and restore it for that one call:

```sh
saved=$GIT_INDEX_FILE
unset $(git rev-parse --local-env-vars)   # intended word-split, needs disable=SC2046
…
GIT_INDEX_FILE="$saved" git add -- <path>
```

Verified git 2.47.3: with it restored, all three commit modes record the staged
change. Caveat for the pathspec mode — the add lands in the temporary index, so
it *is* in the commit but not in the real index, and the path still shows as
modified afterwards. That is the documented partial-commit artifact rather than
a fault in the hook.

**Testing this needs a real commit.** Running `bash hooks/pre-commit` from a
shell sets none of the leaked variables and never exercises the index handoff at
all, so a staging bug is invisible to it and the suite stays green while
production fails. Two tests are needed: one that exports the leaked environment
— the index-breaking trio *and* a bogus `GIT_COMMON_DIR` and
`GIT_OBJECT_DIRECTORY` — asserting the hook exits 0, the `index file open
failed` string is absent, and no refs landed in the bogus common dir; and one
that drives a real `git commit` in each of the three modes.

Related: [[test-the-invocation-path]].
