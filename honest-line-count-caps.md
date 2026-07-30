---
name: honest-line-count-caps
description: "plan/PR line caps mean wrap at ~72 cols then split, never cram long lines; on source files, never shave docstrings to fit — surface the split instead"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 4d8b323c-53e6-4e07-ac83-0926de941a67
  modified: 2026-07-20T15:22:55.455Z
---

When an artifact has a line-count cap (e.g. the ghmem spec's 400-line
cap on plans and PRs), the cap is a proxy for *reviewable*. Hard-wrap
prose at the repo's ~72-col convention first, THEN measure; if it
exceeds the cap, split into multiple focused documents. Never keep the
count down by cramming content onto 150–300-char single lines.

**Why:** my human partner caught exactly this ("you are clearly
cheating on the line count metric") — long unwrapped lines game the
number while destroying reviewability, the thing the cap protects.

**How to apply:** wrap honestly, measure with `wc -l`, split on a
clean seam if over (the ghmem corpus plan split three ways on its
local-vs-network seam). Splitting is explicitly sanctioned by such
caps ("anything longer gets split"). Preserve all information across
the split — duplicated front matter (constraints, shared types) is
fine; dropped content is not. See [[github-body-no-hard-wrap]] for the
opposite case (GitHub bodies render markdown, so no hard wrap there).

Cramming has a **second form** beyond long lines: conveying a full
interface as a run-on paragraph (semicolon-chained methods, comma-run
type lists). One contract per line — see
[[genuine-red-not-missing-sut]].

**Third form of cramming, on SOURCE files: shaving docstrings to fit.**
Fixing the ghmem `status` two-population bug pushed `store.py` from 389
to 406 against the 400-line `check_line_limits.sh` cap, and I trimmed
the two new docstrings until it landed at exactly 400 — declining to
decide the `heat_query.py` split because it sat on the open-decisions
list. Trimming *was* the conservative call there, but it is borrowing
against the split, not an alternative to it: prose is the cheapest
thing in the file to cut and the first thing a later reader needs, and
a file parked at exactly its cap has stopped emitting the signal the
cap exists to emit — the next line added anywhere breaks the build,
which reads as a tooling failure rather than as "this module is due to
be split". If trimming is the only way to fit, say so out loud and put
the split in front of the human as a decision; never let the cap be
satisfied quietly by prose loss.

**Splitting is not free, so the cap is not absolute** (my human
partner, 2026-07-16, on the ghmem Part 3 sync plan): the cap is a
*reader-context* budget, and a split imposes its own reader cost —
re-reading several docs, chasing cross-references between them. When
that cost would exceed what the overage saves, accept a bounded overage
on one cohesive doc rather than split reflexively. The Part 3 plan sat
~40 lines over 400 and was deliberately kept whole: the content was all
validated and the review wasn't unwieldy, so splitting would have been
*more* human effort, not less. This is a judgment call, not a licence
to cram — the honest-wrap discipline above still holds; only the
"always split when over" reflex is tempered.
