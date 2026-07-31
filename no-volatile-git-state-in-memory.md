---
name: no-volatile-git-state-in-memory
description: "Never record commit ids, branch tips, or uncommitted-file state in memory — volatile, stale after the next commit/rebase"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: d05edff3-a464-4040-a692-05eca58470c1
---

Do not put volatile git state in memory files: commit ids (especially
branch/PR tips), "uncommitted as of ..." notes, working-tree status, or
anything invalidated by the next commit, rebase, or squash. Upstream-main
commit ids are technically stable but still discouraged — prefer none.

**Why:** my human partner's branches get rebased and squashed constantly;
a recorded sha is wrong almost immediately and misleads later sessions
more than it helps.

**How to apply:** identify state by durable names — branch names, file
paths, issue/PR numbers, tags, described content ("6 commits, 16
scenarios") — never by sha. If a session needs exact volatile state, it
belongs in a handoff, not in memory. Related: [[always-write-drafts-to-file]]
