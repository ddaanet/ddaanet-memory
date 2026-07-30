---
name: spec-contract-size-predicts-pr-size
description: the 400-line PR cap is soft; a spec task whose contract exceeds ~400 lines predicts its implementation overshoots too — split at spec-review time
metadata:
  node_type: memory
  type: feedback
  originSessionId: d015e0f1-8fa3-4170-9152-2c9265c45dfc
---

The 400-line cap on human-reviewed artifacts (plans, PRs) is a **soft**
guideline for reviewability, not a hard gate — don't block a commit or
agonize over exceeding it in local self-review.

Empirical from ghmem Part 3 (2026-07-17): when a single spec/plan task's
**contract** runs past ~400 lines, its implementation almost certainly
overshoots the 400-line PR cap too — T2's ~400-line contract produced a
1385-line implementation. Contract size and implementation size correlate.

**Why:** catching the overshoot at spec-review time is cheap; discovering
it at PR-slicing time is expensive re-work.

**How to apply:** during spec/plan review, treat a single task's contract
approaching 400 lines as the signal to **split that task** — weight
contract length as a predictor of implementation length in the
update-vs-split decision, don't judge the contract's size in isolation.
See [[honest-line-count-caps]], [[ghmem-project]].
