---
name: gate-cache-must-cover-every-check
description: "A cached quality gate must hash everything its widest check reads, or a failure lands silently and surfaces on an unrelated commit much later"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 22ff8b16-eedb-4ef3-9a9c-5a7a83d05f4f
  modified: 2026-07-26T10:55:30.993Z
---

When a gate caches its verdict against a hash of its inputs, that hash
has to cover the union of what **every** check reads — not the paths most
of them read. Any check whose scope exceeds the hash can be turned red by
a file the hash ignores, and the redness then appears at whatever
unrelated moment something else finally invalidates the cache.

ghmem's `just green` had exactly this hole. `gate-inputs-hash` hashes
`justfile`, `pyproject.toml`, `uv.lock`, `scripts`, `src`, `tests`.
`docformatter`, `mypy` and the line-limit check are all scoped to `src`
and `tests`, but `ruff check -q` took **no path argument**, so it linted
the whole project. Calibration scripts landed under `tools/ghmem/
calibration/`, failed ruff with 31 errors, and never invalidated the
cache — so the commit that introduced them reported green, and the
failure surfaced days later on a one-line edit to `heat.py`.

**Why:** a cached pass is a claim that the checks would still pass. When
a check can read outside the hash, that claim is false, and the delay
between cause and symptom is unbounded — the person who sees the red did
not write the code that broke it.

**How to apply:** for each check in a gated recipe, ask what paths it
actually reads, and reconcile that against the hash. Fix it by narrowing
the check's scope to match the hash (add the path argument, or exclude
the tree) rather than by widening the hash — a broader hash means more
spurious invalidation, and a check running over trees nobody intends to
hold to that standard was the real error. Excluding is right when the
tree is deliberately not library code: ghmem's `calibration` joined
`plans/prototypes` in ruff's exclude list.

**Widening is right when the check is new and its inputs genuinely
belong to the package.** Adding bats suites over `hooks/` to `green`
meant adding `hooks` to the hash and `bats --version` beside
`docformatter`'s: narrowing was not available, because the whole point
of the check is to read that tree. The test for which fix applies is
whether you want the check to see those files at all — not whether the
hash gets broader.

Whichever way it is fixed, **prove the coverage** instead of reasoning
about it: run the gate twice to see `✓ Cached`, append a throwaway
line to a file in the newly-covered tree, confirm the next run does
real work, then remove it and confirm `git diff` on that file is
empty. A hash you only reasoned about is the same claim that failed
here.
