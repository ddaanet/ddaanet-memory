---
name: release-review-slicing
description: "fanning out `edify:release`'s Fable `/code-review high main...dev` over file slices: each slice writes the forked review's out-of-slice findings to a file instead of dropping them, and a slice stays near 20 files or under about 150k agent tokens"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 242f7b47-bca8-40e2-94df-ad348f21f3f1
  modified: 2026-11-18T14:02:31.816Z
---

When a release delta is too large for one Fable reviewer, `edify:release`'s
step-3 code review is split into slices by file, each slice grouping a script
with its tests, its skill and its doc node so a cross-file contradiction lands
in one slice.

- **Each slice keeps every finding the forked `/code-review` returns.** The
  review runs over the whole range and reports on files outside the slice;
  a brief that says "report only on slice files" makes the agent discard
  those. In the gitlore 0.13 release, two of five highs (the session line
  naming `/gitlore:push` beside a mid-merge report; the up-compose path
  emptying the root index on a producer failure) were in the discarded
  set and surfaced only when the slices were asked for them afterwards.
  The brief names a second file, `review-<sha>-<slice>-outside.md`, for
  the out-of-slice findings verbatim, and the triage dedupes them against
  the other slices.
- **Slice size, one data point** (gitlore 0.13, 97 files, 5352 insertions):
  slices of 18, 20, 27 and 28 files used 176k, 165k, 213k and 242k agent
  tokens and ran 17 to 57 minutes, about 8k tokens per file. A slice near
  20 files keeps the agent under the ~150k context where `edify:dispatch`
  says to respawn; one agent over the whole delta would have needed
  roughly 800k.
- **The union of the slices is checked mechanically** against
  `git diff --name-only main...dev`, with the exclusions named (TASK.md,
  TODO*.md, the memory gitlink).

**Why:** my human partner asked for the fan-out and then for the discarded
findings to be kept, and wants the slicing rule in `edify:release` once the
size threshold has more grounding than one run (brief in edify's inbox,
2026-11-18).

**How to apply:** this is the interim home; when `edify:release` carries the
slicing step, delete this file.
