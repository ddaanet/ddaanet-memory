---
name: gate-cache-must-cover-every-check
description: "adding a check to a cache-gated recipe, or splitting a cheap check out of a slow gate; reconcile each check's paths against the hash (narrow/widen/split, then prove it) or it goes red silently on an unrelated commit; a submodule-gitlink input can't be hashed — run that check uncached, ahead of the sentinel"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 22ff8b16-eedb-4ef3-9a9c-5a7a83d05f4f
  modified: 2026-08-11T21:14:52.705Z
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

**A third fix, when the costs inside one gate differ by orders of
magnitude: split.** Give the offending check its own recipe and its own
sentinel, and make the expensive gate depend on it. gitlore's `precommit`
runs a 7m30s bats suite over an input set that deliberately excludes
`agents/`, `commands/` and `skills/`; widening it would have made every
prose edit to a skill body pay the full suite, and narrowing was
unavailable because reading those directories is the distribution
suite's whole job. `check-distribution` runs that one 2s suite behind
its own hash and hangs off `precommit`. The test for this fix is a cost
asymmetry *inside* one gate: a cheap check that reads outside the hash
does not belong in the same cache as an expensive one.

Two constraints on the split. Declare the new gate's inputs as
everything **its** check reads, not merely the paths the other gate
misses — the narrow version is correct only while the other set stays as
drawn, and nothing states that dependency. And it must be its own
recipe, not a second block in the same body: `check-sentinel` sets shell
variables that `record-sentinel` reads back, so a second call in one
shebang clobbers the first's state.

**A fourth case defeats all three: an input that is a submodule gitlink.**
Narrow, widen and split all assume the hash *can* see the tree. It cannot
here — `git ls-files` yields the submodule's one path and nothing to `cat`,
so the recorded sha moves only when the submodule is committed, and every
edit inside it is invisible. gitlore's memory-hygiene checker reads
`memory/`, a gitlink: cached behind the `precommit` sentinel it would have
been skipped on precisely the commits that edit a memory file. The fix is
to run the check **uncached, ahead of the sentinel short-circuit** — cheap
enough (0.4s against a 7m30s suite) that there is nothing to save.

Note the failure runs the *opposite* direction from the one above. An
under-covered hash on an ordinary tree goes red at an unrelated moment,
which is merely confusing; an un-hashable input goes **green on the very
commit the check exists for**, and nothing ever surfaces it.

Whichever way it is fixed, **prove the coverage** instead of reasoning
about it: run the gate twice to see `✓ Cached`, append a throwaway
line to a file in the newly-covered tree, confirm the next run does
real work, then remove it and confirm `git diff` on that file is
empty. A hash you only reasoned about is the same claim that failed
here.
