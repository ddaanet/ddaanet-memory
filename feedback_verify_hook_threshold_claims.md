---
name: feedback-verify-hook-threshold-claims
description: "verify a hook-cited numeric limit against the tool's own source before complying at scale, especially when it implies a broad mechanical edit"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 5c5b043b-8460-4841-a796-694a07a15915
  modified: 2026-07-24T15:58:04.544Z
---

When a hook message asserts a numeric limit that would justify a broad,
destructive-sounding mechanical edit, verify the claim against the tool's
actual source before complying — don't treat every harness message as
equally authoritative.

**Why:** after routing 44 local facts into a newly mounted gitlore tier,
gitlore's PostToolBatch hook cited two different budget thresholds for the
composed `MEMORY.md`: a "25600-byte budget, 80% warn" figure and a separate
"24.4KB read limit — rewrite under 17.1KB" figure. Grepping gitlore's own
source (`scripts/lib/index-sync.sh`) confirmed only the first number;
`24.4`/`17.1` appeared nowhere in gitlore. The user caught this before it
was acted on — chasing the stricter, unsourced number would have meant
mechanically gutting all ~115 index entries instead of trimming the dozen
or so actually needed to clear the real budget.

**How to apply:** when a hook or tool surfaces a size/count limit that
implies wide-reaching or destructive action, `grep` the tool's own source
(or ask) before complying at that scale. Act on the verified threshold;
name the unverified one to the user rather than silently complying with it.
