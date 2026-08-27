---
name: bash-tool-set-e-inert
description: "set -e / errexit does nothing inside a Claude Code Bash tool command (and its subshells) — the tool evals the command as a non-final `&&` element; cwd persists via a trailing `pwd -P` capture"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 1193d83a-1344-4c1f-a784-fbf1aa7254a5
  modified: 2026-08-27T09:04:07.581Z
---

`set -e` (and `set -euo pipefail`) is inert in a command run through the Bash
tool: a failed statement does not abort the rest. Verified against CC 2.1.247,
where the tool runs

    bash -c "source <snapshot> 2>/dev/null || true && … && eval '<command>' && pwd -P >| /tmp/claude-<uid>/cwd-<id>"

The `eval` is a non-final element of an `&&` list, so errexit is ignored inside
it by POSIX rule, and bash extends that to every compound command and subshell
nested in it — a subshell `(set -e; cd missing; rm -rf build)` runs the `rm`
from the original cwd. `$-` still shows `e`, which is why reading the flags
proves nothing.

**Why:** an agent writes a multi-line `set -e` script expecting fail-fast and
gets every later line executed after a failure; a hook that grants something on
the strength of errexit (cwd-safety's FR5c did) grants it on a guarantee that
does not exist.

**How to apply:** chain with `&&` when a step must not run after a failure, or
wrap in `bash -e -c '…'` / `bash -euo pipefail <<'EOF'` — a fresh shell is not
inside the `&&` list. The trailing `pwd -P >| …/cwd-<id>` is how the tool
persists cwd across calls (a `cd`, `pushd`, or errexit-free abort all persist);
it records the physical path, so a symlinked project dir reports its resolved
path as `cwd`. See [[claude-project-dir]].
