---
name: reference_gate_hash_sandbox
description: "a tree-hash gate built on `git add -A` into a throwaway index goes permanently inert under the CC sandbox — it can neither skip nor record; enumerate explicit paths with `git ls-files` instead"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 40ca6ee7-236b-45e8-9428-2f7bbdd25844
  modified: 2026-07-26T14:40:28.396Z
---

A gate that caches "this tree already passed" needs a content hash of the
inputs. The obvious implementation — copy the real index to a temp file,
`GIT_INDEX_FILE=$tmp git add -A`, then `write-tree` — **cannot work under the
Claude Code sandbox**:

```
error: .bash_profile: can only add regular files, symbolic links or git-directories
fatal: adding files failed
```

The sandbox surfaces phantom home dotfiles (`.bash_profile`, `.bashrc`,
`.zshrc`, `.idea`, `.vscode`, …) inside the repo working directory — see
[[reference_sandbox_effects]]. They are not regular files, so `add -A` refuses
outright and the hash comes back empty.

**The damage is the second-order effect, not the first.** A well-written gate
fails open — no hash, no skip, run the gate — which looks harmless. But the
same emptiness also suppresses the RECORD step, so the sentinel is never
written. The gate therefore *never* skips and *never* records: it is not
degraded, it is inert, permanently, and every run pays full price. The symptom
is a sentinel file whose mtime is days older than several green runs.

Worse, the diagnostic goes to **stderr**, which is invisible unless the caller
captures output — so a full suite can run twice under a dead cache with nothing
on screen to say so.

**The fix is to not build an index at all.** Enumerate the paths that actually
gate the outcome and hash their names and contents directly:

```sh
git ls-files -z --cached --others --exclude-standard \
        justfile pyproject.toml uv.lock scripts src tests hooks \
    | sort -z \
    | while IFS= read -r -d '' file; do
          printf '%s\n' "$file"
          if [ -f "$file" ]; then cat "$file"; fi
      done | cksum
```

`--exclude-standard` drops ignored files, `--others` still catches brand-new
untracked ones (a new test file is exactly when a stale cached pass hurts most),
and naming explicit paths means a phantom dotfile at the repo root is never
enumerated. Hash names alongside contents so a rename or deletion counts as a
change. Add the interpreter and any unpinned tool versions to the same stream —
they are gate inputs too.

This is both simpler and strictly more correct than the index dance, which
existed only to reuse git's content-addressing. Reference implementation:
`micro/tools/ghmem/justfile`, `gate-inputs-hash`.

Related: [[reference_sandbox_effects]],
[[feedback_observability_not_gated_on_happy_path]] — a cache that silently
stops caching is exactly the failure nobody is told about.
