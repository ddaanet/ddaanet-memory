---
name: spec-contract-size-predicts-pr-size
description: "the 400-line PR cap is soft; a task's contract size predicts its impl overshoot"
metadata:
  node_type: memory
  type: feedback
  originSessionId: d015e0f1-8fa3-4170-9152-2c9265c45dfc
  modified: 2026-08-10T19:00:57.206Z
---

The 400-line cap on human-reviewed artifacts (plans, PRs) is a **soft**
guideline for reviewability, not a hard gate — don't block a commit or
agonize over exceeding it in local self-review.

Contract size predicts implementation size: when a single spec/plan task's
**contract** runs past ~400 lines, its implementation almost certainly
overshoots the 400-line PR cap too. Observed at roughly 3.5× — a ~400-line
contract producing a 1385-line implementation (ghmem, 2026-07-17).

**Why:** catching the overshoot at spec-review time is cheap; discovering
it at PR-slicing time is expensive re-work.

**How to apply:** during spec/plan review, treat a single task's contract
approaching 400 lines as the signal to **split that task** — weight
contract length as a predictor of implementation length in the
update-vs-split decision, don't judge the contract's size in isolation.
See [[honest-line-count-caps]].
