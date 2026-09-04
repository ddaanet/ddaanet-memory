---
name: design-doc-no-situational-state
description: "a byte count, \"currently over the limit\", a queue depth: delete the figure, keep the mechanism and the pass that acts on it; retire an entry that only reports one, and leave the FR/D/L id gap rather than renumbering"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a0cb79ca-c935-4c91-8ed2-3e3a5526d159
  modified: 2026-09-02T22:12:15.776Z
---

A living design doc records what the system is and why. A **measurement of
mutable state** is neither: a byte count of a file that changes on every write,
a "this is currently over the limit" claim, a queue depth, a count of open
findings. These read like the grounded dates and figures that *should* stay
("verified against CC 2.1.150", "measured over 76 bullets") but they describe a
moment rather than supporting a claim.

**Why:** they go stale between the commit that records them and the next one,
and refreshing the figure does not fix the category error — it schedules the
same correction again. A stale number in a document whose whole authority is
being current truth costs more than the number was ever worth. Measured
instance: `docs/design.md` L-5 recorded `memory/MEMORY.md` at "~28.9 KB against
a 24.4 KB limit"; the store was 31.7 KB three weeks later, and the fix was
queued as a review finding as though the figure were the defect.

**How to apply:**

- **Delete the figure, keep the mechanism and the pass.** "gitlore's index
  budget is a real loader cutoff rather than a soft limit, so retiring entries
  is a periodic pass (`/gitlore:index-audit`)" survives every future
  measurement; "the index is 31.7 KB and truncated" does not.
- **State that has to live somewhere lives where state lives** — the doc's
  status stamp (§1 Now), a task frame, a report. Those are read as of-a-moment
  by construction; the design record is not.
- **An entry existing only to report that a mutable quantity is out of bounds
  gets retired, not corrected.** Its content was always a task, and the task
  belongs to whatever pass owns it.
- **Retire, don't renumber.** FR/D/L ids are cited from changelog entries,
  plans, reports and other memory files, so removing one leaves a gap in the
  sequence. Renumbering silently repoints every existing citation past the gap
  at the wrong entry.

Related: [[design-doc-writing]] for the section structure and the
changelog-vs-design split this refines; [[gitlore-tier-index-budget]] for the
loader cutoff itself, which is the mechanism the retired example pointed at.
