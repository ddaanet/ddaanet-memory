---
name: preflight-excludes-memory-submodule
description: "` M memory` in the parent's `git status --porcelain` is the resting state, not a dirty tree: a release-readiness gate like `ddaa:preflight` must not abort on it, since the parent's pre-commit hook commits and gates memory into the release commit; other dirty paths still block"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 39cc1769-caeb-45dd-ad0d-0976db718362
  modified: 2026-08-09T18:43:36.102Z
---

In a repo whose memory is a gitlore submodule, a dirty memory store is the
resting state between commits, not an incomplete working tree. The parent's
`git status --porcelain` reports ` M memory`, and the store itself carries
staged content, for as long as facts have been written and no parent commit has
followed.

**Why:** nobody commits inside the memory store. The parent repo's pre-commit
hook records the store, gates it, and stages the moved gitlink into whatever
parent commit comes next. The dirty state is what the design produces on
purpose, and it clears as a side effect of that commit rather than needing an
act of its own.

**How to apply:** a clean-tree assertion, or a "submodules must have no
uncommitted changes" rule, has to exclude the memory submodule's path and report
it as information. A release-readiness gate that aborts on it — `ddaa:preflight`
does — is unpassable until someone performs by hand a commit the release was
going to make anyway. The exclusion covers that one path: untracked and modified
files elsewhere in the parent are genuine findings, and a gate that waves those
through is broken in the other direction. The one real precondition dirty memory
carries is a fresh approved summary before the next parent commit — written
after the last memory edit, since the gate compares mtimes — or that commit
blocks and tells you so. That is a commit-time requirement, not a readiness
verdict.
