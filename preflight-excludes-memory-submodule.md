---
name: preflight-excludes-memory-submodule
description: "` M memory` in the parent's `git status --porcelain` is the resting state, not a dirty tree: a release-readiness gate like `ddaa:preflight` must not abort on it, since memory waits for its own approved commit; other dirty paths still block"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 39cc1769-caeb-45dd-ad0d-0976db718362
  modified: 2026-08-09T18:43:36.102Z
---

In a repo whose memory is a gitlore submodule, a dirty memory store is the
resting state between memory commits, not an incomplete working tree. The
parent's `git status --porcelain` reports ` M memory` for as long as facts have
been written and no memory commit has followed.

**Why:** memory is committed only by its own act: the agent shows a summary of
the changed facts, the user approves it, and the `gitlore:commit` skill commits
the store and stages the moved gitlink. No hook does it on the side of a parent
commit, so dirty memory waits — across parent commits, releases included —
until that approval happens.

**How to apply:** a clean-tree assertion, or a "submodules must have no
uncommitted changes" rule, has to exclude the memory submodule's path and report
it as information. A release-readiness gate that aborts on it — `ddaa:preflight`
does — makes the release wait on a memory approval that is a separate decision.
The exclusion covers that one path: untracked and modified files elsewhere in
the parent are genuine findings, and a gate that waves those through is broken
in the other direction. Whether to commit the pending memory before the release
is the user's call, raised as information, not a readiness verdict.
