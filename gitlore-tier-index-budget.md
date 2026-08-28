---
name: gitlore-tier-index-budget
description: "mounting a populated tier imports its whole index; composition orders tier-first, so the consumer's OWN project lines fall past the 24.4KB load cutoff"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 76b63f5f-eb97-4886-b76a-f8cd871faac9
  modified: 2026-08-14T23:05:05.155Z
---

`/gitlore:add-tier` on an already-populated tier copies every one of its
index lines into the consuming repo's root `MEMORY.md`. A tier carrying ~150
facts at the ~230 bytes an informative index line costs lands as ~33KB of
index in a repo that may have had 14KB of its own.

Two things make the overflow worse than a size warning:

- **Composition puts the tier's pointer block ahead of the project's own
  lines.** The loader truncates the tail at 24.4KB, so what goes invisible
  first is the consuming repo's own project memory — the lines most specific
  to the work happening there.
- **The fix is not local.** Trimming project lines cannot recover 16KB when
  the tier block alone is 33KB. Getting under the limit means shortening the
  shared tier's lines, which changes what *every* repo mounting that tier
  loads. That is a decision for the tier's owner, not for whichever repo
  happens to notice, and index compaction is exactly where triggers get lost
  (`/gitlore:index-audit` is the pass that audits for it).

**How to apply:** measure before mounting —
`grep '^- \[' MEMORY.md | wc -c` in the tier's own store against the budget
already spent in the target repo. If the sum clears 24.4KB, the mount is
still correct but it lands with a known-truncating index; say so at the time
rather than letting a later session discover its project facts silently
missing. Related: [[gitlore-tier-merge-direction]]; the tier test itself is in
gitlore's `memory-writing` skill.
