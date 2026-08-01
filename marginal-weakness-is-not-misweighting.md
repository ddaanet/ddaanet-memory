---
name: marginal-weakness-is-not-misweighting
description: "a metric can discriminate at the base rate alone and still earn a heavy weight"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 22ff8b16-eedb-4ef3-9a9c-5a7a83d05f4f
  modified: 2026-07-25T11:50:20.518Z
---

Sorting a labelled sample by one feature at a time and reading off how
much signal lands in the top half measures **marginal** discrimination.
A weight in a composite score is a **conditional** quantity — what that
feature adds once the others are known. The two come apart, and the gap
runs in the direction that gets a useful feature deleted.

ghmem's `related_items` is the worked example. At n=54 it sorted signal
into the top half at exactly the 41% base rate, which reads as a coin
flip, and it carried the heaviest weight in `HEAT_WEIGHTS` (3.0) — an
apparently damning pair. At n=198 its fitted coefficient came out +0.243,
second largest in the vector, and zeroing it *cost* corpus-weighted AUC
(0.818 against 0.843). It remains the weakest lone ranker (AUC 0.595).
Both facts are true at once.

**Why:** a feature can be uninformative alone and informative in
combination — it separates cases the other features tie, or it corrects
them where they mislead. A one-feature-at-a-time table cannot see that,
because it never conditions on anything.

**How to apply:** to judge a weight, fit the whole vector and compare
rankings, never a per-metric sort. Before cutting a weight, check what
dropping it costs the full score — if the answer is "nothing", that is
the evidence; a weak marginal table is not. And when a sample is
stratified on the composite, its single-metric-versus-composite
comparison is handicapped by construction, so treat any "the part beats
the whole" result from it as an artifact until a differently drawn sample
reproduces it. See [[ground-formulas-in-data]] and
[[provisional-values-not-provisional-code]].
