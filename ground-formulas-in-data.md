---
name: ground-formulas-in-data
description: "derive from corpus analysis, never hip-shooting"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 2eeaac7d-8f74-4a8c-941d-66cd37826d27
  modified: 2026-08-10T19:00:54.642Z
---

Never ship hip-shot heuristic constants. Any scoring formula, weight,
or threshold must be grounded in analysis of the actual data it will
run on, with the analysis recorded.

**Why:** a constant chosen by intuition encodes a guess as a
specification, and nothing downstream distinguishes it from a measured
one. A plausible-looking weight (`comments + reactions + 2*participants`)
and a round threshold (3 PRs, 10 commits) survive review precisely
because they look deliberate.

**How to apply:** before fixing constants, run a cheap distribution
analysis over the real corpus (percentiles, known-ground-truth
calibration — e.g. do known maintainers classify as maintainers, do
known-hot threads rank hot?), record it, and derive the constants from
it. Keep them tunable, but tuned from evidence.
