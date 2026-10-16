---
name: git-protocol-file-allow
description: "`fatal: transport 'file' not allowed`; the child *clone* reads it, so repo config can't fix it — `git -c`"
metadata: 
  node_type: memory
  type: reference
  originSessionId: d7a4fa69-e3f9-4102-b9d6-7d3976d65bcf
  modified: 2026-07-22T14:43:42.835Z
---

`git submodule add -- /path/to/bare.git name` fails with:

```
fatal: transport 'file' not allowed
```

`protocol.file.allow` defaults to `user`: allowed for a command the user typed,
blocked for one git spawns on their behalf. `submodule add` clones via a child
process, so it is the blocked case. Same for `clone --recurse-submodules` and
`submodule update`.

**Setting it as repo config does not work.** `git -C parent config
protocol.file.allow always` has no effect, because the policy is read by the
*clone* — a fresh process, in a repository that does not exist yet, which never
reads the parent's config. The fix has to travel through the environment or the
command line:

- `git -c protocol.file.allow=always submodule add …` when you own the command
  line. `-c` propagates to child processes via `GIT_CONFIG_PARAMETERS`.
- `GIT_CONFIG_COUNT=1 GIT_CONFIG_KEY_0=protocol.file.allow
  GIT_CONFIG_VALUE_0=always` when you do not — a test harness, or an agent
  session whose git calls are made by production code you are exercising rather
  than editing. Export it once and it reaches the whole process tree.

The env form survives a git hook's `unset $(git rev-parse --local-env-vars)`:
that list carries `GIT_DIR`, `GIT_WORK_TREE`, `GIT_INDEX_FILE` and friends, but
no `GIT_CONFIG_*` (the `shell-scripting:shell-gotchas` skill, `references/environments.md`).

Loosening it is a **test-fixture** move, not a production one — it is the same
transport-policy dial as [[git-ext-transport]], and `file` is on that
list for a reason. Reach for it when local bare repos are the only remotes
available (a sandbox with no network), and scope it to the harness.
