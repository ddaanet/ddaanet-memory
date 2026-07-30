---
name: droplet-cpu-variability
description: "the shared ddaanet dev droplet is a virtual server David resizes between 1 and 2 vCPUs to save cost when idle — currently 2"
metadata: 
  node_type: memory
  type: project
  originSessionId: 4a70b2ed-1f70-4f5b-8a35-b367c5f011c4
  modified: 2026-07-28T18:35:00.450Z
---

The dev droplet used for ddaanet repo work (including gitlore) is a virtual
server. David resizes it down to 1 vCPU to save cost when he isn't using it
much, and back up to 2 when active — it's currently 2 vCPUs (2026-07-28).
This came up while scoping a `bats --jobs`-only test-suite speedup as
insufficient on its own — see
[[perf-fix-check-parallelism-assumption]].

**Why:** David corrected an earlier draft of this memory that framed the
CPU count as flaky/unreliable hardware — it's a deliberate, known resize he
controls for cost, not something to treat as unpredictable. It also applies
to any repo run on this droplet, not just gitlore.

**How to apply:** don't design a test/build-gate speedup, on this droplet or
any other repo hosted on it, whose entire benefit disappears at 1 vCPU (the
resized-down state is a real, expected operating mode, not an edge case) —
but don't skip a parallelism win either, since 2 vCPUs is also real and
current. Do both.
