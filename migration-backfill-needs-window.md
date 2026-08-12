---
name: migration-backfill-needs-window
description: "adding a NOT NULL field to an event-sourced model via \"backfill by event, not a rollout flag\" — verify the backfill can actually run against real legacy data before the reducer enforces the field"
metadata: 
  node_type: memory
  type: project
  originSessionId: d9e6be72-a5b0-427c-86b4-d3be817e2e3a
  modified: 2026-08-12T10:35:39.885Z
---

A NOT NULL field added to an event-sourced model via "backfill by event, not
a rollout flag" only works if the backfill can run *before* the reducer
enforces the field unconditionally on every event of that type. If the
reducer has no tolerance for a missing field, and real (non-fixture) legacy
data predates the field, there is no window: the backfill needs a working
derived-store read (a `Store.routes()`-style query) to enumerate existing
rows, which needs a rebuild that replays the ledger, which crashes on the
very legacy events lacking the field before the backfill ever gets to run.
Appending a correction event afterward cannot fix this either — sequential
replay dies on the original un-migrated line before it ever reaches a later
correction for the same row.

**Why:** ghmem's `Route.item_updated_at` migration
(`docs/superpowers/specs/2026-08-11-ghmem-d-reentry-design.md`'s Rollout
section) assumed "append one corrected event per row, then rebuild" would
work. That only holds if the append happens before the reducer's strict
requirement ships in the code actually being run. Once the schema-change
task and the backfill-script task land in the same branch, that window never
existed against the live corpus — only against test fixtures, which never
model a genuinely un-migrated legacy event because a fixture always seeds
routes with the field already present.

**How to apply:** when a plan pairs a NOT NULL schema change with a one-off
backfill script, verify — before writing the backfill task, not after
merging — that the backfill can run against real production data exactly as
proposed: check whether the target store/reducer is reachable in a state
that has the new requirement but old data. If it isn't, the working fix is
an in-place edit to the persisted event log itself (not an appended
correction event), done once by hand with a diff reviewed before commit —
same mechanical, no-judgment-needed character an append-based backfill was
supposed to have, just structurally different because append cannot survive
a single-pass, order-sensitive replay.
