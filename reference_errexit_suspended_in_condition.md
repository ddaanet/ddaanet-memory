---
name: reference_errexit_suspended_in_condition
description: "`set -e` is off inside anything reached from a condition — `if x=$(f)`, `while`, `&&`, `!` — including subshells f spawns, so a mid-function failure neither aborts nor shows; fail each step explicitly with `|| exit 1` / `|| return 1`"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 40ca6ee7-236b-45e8-9428-2f7bbdd25844
  modified: 2026-07-26T18:16:44.116Z
---

Bash disables `errexit` for the entire command whose exit status is being
tested — the condition of `if`/`while`/`until`, either side of `&&`/`||`, and
the operand of `!`. The suspension is not lexical: it reaches into command
substitutions and subshells that command spawns.

So a function invoked as `if out=$(f); then` runs its **whole body** without
errexit, however carefully `set -euo pipefail` was written at the top of the
script. Only `f`'s final exit status is consulted; a failing step in the middle
is neither fatal nor visible.

**The shape that hides it.** A gate hashed its inputs with

```sh
gate-inputs-hash () {
    {
        bats --version          # <- fails silently
        git ls-files -z ... | ... | while read ...; done
    } | cksum
}
```

called from `if hash=$(gate-inputs-hash); then`. With `bats` broken, the
version line simply dropped out of the stream: the group's status is its
*last* command's, that one succeeded, `cksum` hashed a shorter stream, and the
gate went on caching against a hash that no longer covered the tool. Nothing
was reported at any layer.

The tell was a test: an "unhashable inputs record nothing" case passed green
because the hash had *succeeded*, not because the failure path worked
([[feedback_mutation_check_negatives]] — the assertion could not distinguish
the two).

**How to apply:** inside anything a condition can reach, do not rely on
errexit for control flow. Fail each step on its own — `cmd || exit 1` in a
subshell, `cmd || return 1` in a function — and keep `pipefail` so a failing
stage still reaches the caller. Note the asymmetry: `pipefail` and `nounset`
are *not* suspended in a condition, only `errexit`, so a pipeline's status
does propagate once its stages report honestly.

Related: [[feedback_observability_not_gated_on_happy_path]] — a check that
silently stops checking is the failure nobody is told about.
