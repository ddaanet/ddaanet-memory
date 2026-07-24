---
name: feedback-verify-review-findings
description: "verify a reviewer/subagent \"HIGH\" finding empirically before acting — escalated style rules masquerade as correctness bugs"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: d88d17aa-b0e8-4172-ae43-273c924cb8cf
---

When a review subagent reports a HIGH / correctness finding that would
trigger broad edits, reproduce the claimed failure with a minimal probe
*before* churning code.

**Why:** in the test-suite conversion, the bats code-quality agent
escalated a style-consistency rule ("guard every non-final assertion")
to a correctness HIGH ("masked failures"). A 5-line bats probe showed
errexit already aborts on those failures ([[reference-bats-errexit]]), so
the finding was false and would have caused a noisy, pointless 36-test
churn. Review agents inherit rubric rules and apply them literally; a
rubric author may flag a property as "consistency" that the reviewer then
reclassifies as a bug.

**How to apply:** for any review finding that implies a sweeping change,
reproduce the failure first; if it can't be reproduced, downgrade it and
say so. Pairs with [[feedback-design-doc-immutability]] (don't trust
write-time claims — re-examine).
