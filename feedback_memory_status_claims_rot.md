---
name: feedback-memory-status-claims-rot
description: "Status claims in memory ('not yet run', 'pending', 'in progress') rot silently because nothing updates them when the state changes — verify against git/disk before repeating one as fact"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 94e0a066-a634-4a25-a32d-b0f53b992c25
  modified: 2026-07-20T17:41:15.578Z
---

Do not restate a memory's **status claim** as current fact without checking it.
Claims of the form "release not yet run", "pending implementation", "still
open" are the ones that go stale, because the event that falsifies them
(running the release, landing the work) does not write back to the memory file.

Incident 2026-07-20: [[project_worktree_handoff_root]] said "`just release`
is the remaining user-driven step — not yet run", so a release was recommended
on the premise that shipped work was untagged. One command —
`git merge-base --is-ancestor 5d0145a v0.9.0` — showed the work was inside
v0.9.0 all along. The recommendation was built on a stale claim asserted
without verification, and the system-reminder on the file had already flagged
it as 41 days old.

**Why:** descriptive memories (how a thing works, why a decision was made) age
gracefully; state memories describe a moving target and are wrong the moment
the state moves. The user pushed back with "explain release nonsense" — the
reasoning did not survive one check.

**How to apply:** when a memory names a status, verify it in the source of
truth before acting — git tags/ancestry for shipped-or-not, the filesystem for
exists-or-not, the issue tracker for open-or-not. It is one cheap command.
Then **fix the memory file in the same turn** so the next session doesn't
repeat it. Prefer writing status with its evidence ("released in v0.9.0,
verified via `git merge-base …`") over bare assertions.

Related: [[feedback_verify_review_findings]] (same reflex, applied to a
subagent's claims rather than a memory's).
