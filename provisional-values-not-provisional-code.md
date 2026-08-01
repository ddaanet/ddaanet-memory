---
name: provisional-values-not-provisional-code
description: "uncalibrated constants: a parameter to bind, or a licence for a placeholder implementation"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: d81694ba-c563-49a4-a0ce-f8029e7ca7fd
  modified: 2026-07-20T13:31:01.631Z
---

A weight or threshold still awaiting calibration does not justify a
placeholder *implementation*. Weights are parameters: bind them into
the query. Only a change in the model's SHAPE (nonlinear score, rank
fusion, a normalization pass) would force a rewrite — "the numbers
aren't final" never does.

Corollary: a per-item Python loop calling one query method per metric
is a reimplemented `GROUP BY`. Check whether the aggregate is
expressible in SQL before writing the loop.

Second corollary: if the consumer of some output is analysis, emit a
data format (CSV with a header, JSONL) — not a dense human line that
has to be regex'd apart. Truncating expensive noise with a `--limit`
is not a fix; it yields less noise while breaking the one consumer
that needs the whole distribution.

**Why:** my human partner caught `ghmem heat` fanning out ~5 queries
per item over a ~3.8k-item corpus to print one bespoke line each, with
no programmatic consumer — and rejected both my "calibrate first, then
optimize" defence and my `--limit` half-measure. The rewrite was one
correlated-subquery aggregate with the weights bound as parameters:
22 statements → 1 on the test fixture.

**How to apply:** when a plan defers an optimization "unless
calibration shows it matters", check whether the deferral is really
about the model's shape. If it isn't, build it properly now — the
calibration consumer is usually the one that most needs it done right.

See [[ground-formulas-in-data]], [[no-code-for-impossible-cases]].
