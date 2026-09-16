---
name: shared-claude-oversize-false-positive
description: "the \"over the 4KB recall limit… rewrite to under 2.8KB\" hook on `shared-claude.md`: it is the `@`-imported conventions tier, never recall-fetched; do not comply"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 4fb6c683-e02f-4bf0-a43d-56cca0cb9f9a
  modified: 2026-09-16T18:05:58.817Z
---

Any edit to `memory/ddaanet/shared-claude.md` trips the gitlore memory-size
hook: *"this write left the memory file at ddaanet/shared-claude.md at \<size\>,
over the 4KB recall limit... Rewrite it to under 2.8KB now."*

**It is a false positive, and complying is destructive.** The 4KB limit exists
because recall shows other sessions only a memory file's first 4KB. This file
is never recall-fetched: each repo's `CLAUDE.md` imports it with
`@memory/ddaanet/shared-claude.md`, so it loads **whole**, every session, in
every repo mounting the tier. The file says so in its own header. Shrinking it
to 2.8KB would strip the shared conventions from every ddaanet repo to satisfy
a cap that does not apply to it.

The hook keys on the path living under the memory store and cannot distinguish
the conventions tier from an ordinary fact.

**How to apply:** make the edit, report the hook firing and why you did not
comply, and move on. Do not split the file, do not create a `-2` continuation,
do not trim conventions to fit. The file's real budget is the one its own
header states — every line is paid for by every session in every ddaanet repo —
which is an editorial discipline, not a byte count a hook can check.
