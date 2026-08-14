---
name: examine-evidence-drift
description: "two co-maintained artifacts diverge, or one is cited as evidence about the other (a sibling, a prior version): establish the relationship from git history, never assumption"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: b4fe98c4-c5d9-4e86-9302-1be83a10806c
  modified: 2026-08-14T20:13:42.034Z
---

When two artifacts are supposed to agree but can diverge — a memory file's `MEMORY.md` index one-liner vs its frontmatter `description`, a spec vs code, a doc vs its implementation — do NOT assume a priori which one is fresher or authoritative. Pull the git history and let the commit pattern show the relationship: which surface a given change actually touched, and when. The same caution covers a cited-sibling comparison ("the mature version lacks this feature"): the sibling's own lineage can make it circular evidence rather than independent confirmation.

**Why:** Two assertions of a source-of-truth direction from plausible reasoning — first "frontmatter is stale," then "the index one-liner is the living copy" — drew the same correction both times: "don't assume either way, examine evidence." A `git log -S/-p/--stat` audit then showed the drift is **bidirectional**: sometimes the index line is fresh and the frontmatter stale, sometimes the reverse. Any assumed direction is wrong a large fraction of the time, and building a sync/regeneration on the wrong assumption silently propagates staleness the wrong way.

**How to apply:** For any "which of these two is right?" question about co-maintained content, run the history on *both* surfaces before designing a fix, and present concrete commits (this file changed here, that one didn't) rather than a theory. Only after the evidence establishes the direction — or that there is no single direction — design the mechanism. The same check applies when an artifact is cited as evidence about another: trace its lineage in history before accepting the comparison as independent.
