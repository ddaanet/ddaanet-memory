---
name: ground-formulas-in-data
description: "derive from corpus analysis, never hip-shooting"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 2eeaac7d-8f74-4a8c-941d-66cd37826d27
---

Never ship hip-shot heuristic constants. Any scoring formula, weight,
or threshold must be grounded in analysis of the actual data it will
run on, with the analysis recorded.

**Why:** my human partner flagged the ghmem heat formula
(`comments + reactions + 2*participants + ...`) and standing thresholds
(3 PRs / 10 commits) as hip-shooting: "always ground heuristic formulas
in data analysis."

**How to apply:** before fixing constants, run a cheap distribution
analysis over the real corpus (percentiles, known-ground-truth
calibration — e.g. do known maintainers classify as maintainers, do
known-hot threads rank hot?), record it, and derive the constants from
it. Keep them tunable, but tuned from evidence.
