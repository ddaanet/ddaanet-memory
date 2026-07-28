---
name: feedback_perf_fix_check_parallelism_assumption
description: "parallelism alone is not a sufficient perf fix — pair --jobs-style concurrency with cutting the actual per-unit work"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 4a70b2ed-1f70-4f5b-8a35-b367c5f011c4
  modified: 2026-07-28T18:35:15.972Z
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
