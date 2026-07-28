---
name: feedback_perf_fix_check_parallelism_assumption
description: "parallelism alone is not a sufficient perf fix — pair --jobs-style concurrency with cutting the actual per-unit work"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 4a70b2ed-1f70-4f5b-8a35-b367c5f011c4
  modified: 2026-07-28T19:54:03.585Z
---

When a task is slow because it repeats an expensive unit of work many times
(e.g. a test suite forking hundreds of git subprocesses per fixture), a
parallelism fix (`bats --jobs`, `make -j`, `xargs -P`) is a legitimate part
of the fix but not sufficient on its own — pair it with cutting the
per-repetition cost itself (cache, template, memoize the repeated work).

**Why:** proposed `bats --jobs` as *the* fix for gitlore's ~600s `just
precommit` runtime; corrected — "parallelism is nice, but it's not enough
on its own" (David, 2026-07-28). The correction was additive (do both), not
a rejection of parallelism in favor of caching. (The dev droplet is a
virtual server David resizes between 1 and 2 vCPUs to save cost when idle —
see [[ddaanet/project_droplet_cpu_variability]] — so a fix that only pays
off at 2 vCPUs loses value whenever it's scaled down, but that's a reason
to also cut per-unit cost, not a reason to skip the parallelism win.)

**How to apply:** when profiling shows the cost is N repetitions of a fixed
unit of work, apply both levers: reduce/cache the per-repetition cost AND
enable whatever concurrency is available. Neither substitutes for the
other.

**Addendum (same session, same fix):** identifying the "N repetitions of a
fixed unit of work" itself from a subprocess-count estimate (grep call
sites, count forks per call) is not profiling — it's a plausible-sounding
model that can be wrong. Implemented fixture-template caching for gitlore's
highest-count helper (`make_parent_with_memory`, ~280 call sites) based on
exactly that kind of estimate; direct benchmark confirmed the helper itself
got ~3x cheaper per call (0.5s → 0.17s), but `just precommit` end-to-end
showed no measurable change (~595s vs ~590-613s baseline) — the helper was
never the dominant cost. Before implementing a targeted perf fix, get a
per-unit timing breakdown (e.g. `bats --timing`, wall-clock before/after on
the *whole* gated command) — a subprocess or call-site count is a hypothesis
to test, not a measurement to act on.

**Resolution (same session):** switching to `bats --timing` found the real
hot spots directly — two `@test`s neither the subprocess-count model nor
the fixture-caching fix had touched turned out to dominate. Fixing those
(details in the commit) cut `just precommit` ~20%, confirmed end-to-end.
Reinforces the same point from the other direction: `--timing` (or
equivalent direct profiling) finds the real target; a subprocess-count
model, however careful, is still a guess until checked against it.
