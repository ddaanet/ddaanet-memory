---
name: test-with-live-data-before-designing
description: "before planning the next subproject: real data, not fixtures"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 2ab4326a-2ac4-4c90-9dd8-4da55efdf552
  modified: 2026-08-01T14:26:10.878Z
---

Before planning a subproject that builds on an existing pipeline, run
that pipeline against real data first. Fixtures are written by the same
hand as the code, so they encode the same assumptions and cannot
disprove them.

**Why:** ghmem had 187 green tests and had never fetched a byte from
GitHub. One live run produced nine findings, and the classes they fall
into are the ones fixtures structurally cannot reach:

- **Environment the fixture always provides.** Every test used
  `tmp_path`, which exists; nothing in the system created `corpus/`, so
  a fresh checkout tracebacked on every read command.
- **Scale.** No index on any foreign key is free at 20 fixture rows and
  made `heat` take >10 minutes at 3875 items (12 seconds once indexed).
- **Distribution.** Heat weights set by intuition about importance were
  inverted in practice, because metric scales differ 40× and weight ≠
  influence. Sparsity too: most items score zero on most metrics.
- **Boundaries real history crosses and fixtures don't.** GitHub's
  nested connections truncate at their first page, clipping the
  highest-weighted metric precisely on the most-connected items.
- **Unreachable code paths.** A whole designed mechanism (solo-item
  diff fitting) turned out to have no trigger in real data.

**How to apply:** do the read half first — it is reversible. Sync,
rebuild, then run every read command over the whole corpus, not one
sampled item. Sweep for errors AND warnings across all items; that is
what closed a property left "hypothesized, not observed" for weeks.
Hold back anything that writes a non-regenerable ledger until the read
half is proven. Measure distributions, not just "it ran" — the p50/p90/
max of every quantity a downstream constant is supposed to bound.

Related: [[tests-pass-for-the-wrong-reason]],
[[ground-formulas-in-data]], [[tests-that-cannot-discriminate]].
