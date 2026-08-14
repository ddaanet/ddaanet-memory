---
name: examine-evidence-drift
description: "two co-maintained artifacts diverge, or \"the mature sibling lacks X\" is cited as evidence: establish drift direction and lineage from git history, never assumption"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: b4fe98c4-c5d9-4e86-9302-1be83a10806c
  modified: 2026-08-14T18:42:22.673Z
---

When two artifacts are supposed to agree but can diverge — a memory file's `MEMORY.md` index one-liner vs its frontmatter `description`, a spec vs code, a doc vs its implementation — do NOT assume a priori which one is fresher, authoritative, or independent. Pull the git history and let the commit pattern show the relationship: which surface a given change actually touched, and when — or where the cited artifact came from.

**Why:** Two assertions of a source-of-truth direction from plausible reasoning — first "frontmatter is stale," then "the index one-liner is the living copy" — drew the same correction both times: "don't assume either way, examine evidence." A `git log -S/-p/--stat` audit then showed the drift is **bidirectional**: sometimes the index line is fresh and the frontmatter stale, sometimes the reverse. Any assumed direction is wrong a large fraction of the time, and building a sync/regeneration on the wrong assumption silently propagates staleness the wrong way. Second instance (2026-08-13): a design review accepted "the mature sibling lacks this feature" as justification for removing that feature — the sibling was itself created by stripping the feature from a copy of the artifact under review, so the comparison was circular; nothing at review time signals such lineage, and my human partner caught it mid-`/proof`.

**How to apply:** For any "which of these two is right?" question about co-maintained content, run the history on *both* surfaces before designing a fix, and present concrete commits (this file changed here, that one didn't) rather than a theory. Only after the evidence establishes the direction — or that there is no single direction — design the mechanism. Likewise when a review cites a sibling artifact as evidence ("the other version doesn't have it"): trace the cited artifact's lineage in history before accepting the comparison as independent.
