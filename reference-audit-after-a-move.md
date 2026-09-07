---
name: reference-audit-after-a-move
description: "a stale-comment sweep scoped to the move under review inherits its blind spot and under-reports; classify every hit of a whole-tree grep against where the symbol lives now"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 92f172d3-be6d-4ab9-a83b-558f8348458a
  modified: 2026-09-07T12:51:40.626Z
---

When a symbol moves between modules and you audit the comments and
docstrings naming its old home, check each reference against where the
symbol actually lives **now** — not against the move you are reviewing.

Scoping the audit to one move inherits that move's blind spot. The
prose that survives a split is exactly the prose nobody re-read, so
older drift accumulates in the same files the new drift lands in, and
a reference that is not stale *because of this move* can still be
flatly wrong.

**Why:** a review of ghmem's `ingest.py` → `routing_reduce.py` split.
Two independent passes — a delegated code review and a cross-cutting
one — each reported 6 stale references and each produced a
leave-alone list carrying the same two errors. An earlier split had
moved the raw-node structs to `raw_nodes.py`, and three further
comments still credited `ingest.py` with them. The real count was 9.
Both passes asked "is this reference stale because of *this* split?",
answered no, and moved on — true, and beside the point.

**How to apply:** make the exit condition a whole-tree grep for the
old module name with **every** hit classified accurate or stale, not a
reading of the diff under review. State the classification per hit;
"the rest are fine" is where the misses hide. Expect the count to grow
past what the review reported, and treat that as the audit working.

The fixer caught all three because writing a correction forces naming
the symbol's real home, where reviewing only asks whether the existing
name is defensible. Where a check can be phrased as "produce the right
answer" rather than "judge this answer", prefer it.

See [[examine-evidence-drift]] on getting drift direction from git rather
than from which artifact looks more authoritative.
