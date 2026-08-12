---
name: tier-routing-and-links
description: "ddaanet vs project memory: does the mechanism depend on this repo, or only on its class (plugin-shaped, hook-managed)?; and the move carries `[[links]]` and repo-relative `docs/` paths verbatim, so ones aimed at a project-local sibling go permanently dead from every other consumer — unlike a write-it-later dangle"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a3e2f110-a3fe-4f99-bfb6-a2343f2a4110
  modified: 2026-08-12T10:54:16.064Z
---

Two halves of one moment: deciding a fact belongs in a shared tier rather
than project memory, and carrying it across without leaving dead pointers
behind. Merged from two facts recorded in sessions
`a3e2f110-a3fe-4f99-bfb6-a2343f2a4110` (routing) and
`c1666e33-a938-42c3-a372-e89af689e7f6` (links).

## Does it belong in the tier?

When a fact looks project-local at first glance — a sandbox quirk, an
environment gotcha, a harness interaction — check whether it's actually
scoped to *this repo's content* or to *the class of repo this is* (a
Claude Code plugin, a git-hooked project, etc.) before filing it under
project memory.

**Why:** during gitlore's memory proof pass, I initially proposed keeping
`reference_own_hooks_json_sandbox_erofs` (a sandbox EROFS on a plugin's
own active `hooks/hooks.json`) as a standalone project-tier fact — it
mentions gitlore's own repo in its evidence. David's correction: "it
should go to ddaanet, it affects anything plugin-shaped." The mechanism
(harness mounts an active plugin config read-only under the sandbox) has
nothing to do with gitlore specifically — it fires for any Claude Code
plugin repo with a live `hooks/hooks.json`.

**How to apply:** ask "does this fact's *mechanism* depend on something
unique to this repo, or just on this repo being a member of a class
(plugin, hook-managed repo, submodule-based memory store, …)?" If it's the
latter, route to `memory/ddaanet/` (or the matching tier), merged into an
existing topic file if one already exists for that trigger surface —
don't leave a same-fact stub in the project tier "just in case."

## The move carries pointers that stop resolving

When a memory is routed from a project's own store up into a shared tier, its
`[[links]]` travel with it verbatim. Any that pointed at a **project-local**
sibling now resolve only in the repo that wrote them, and are dead from every
other consumer of the tier. Strip those pointers as part of the move.

**Why:** mounting the `ddaanet` tier in micro routed 48 facts up. Sixteen of
them carried `[[ghmem-project]]`, `[[micro-colwrap-project]]`,
`[[worktree-handoff-root]]` and similar. Nothing was authored wrong — they were
provenance citations ("**Why:** during ghmem subproject 3
([[ghmem-project]])", "confirmed twice on ghmem's `sync.py`") and they resolved
correctly while everything lived in one flat directory. The *move* created the
boundary underneath prose that predated it. This is not the blessed
write-it-later dangling link: the target is deliberately staying local, so the
link never becomes valid, and every project accumulates its own local facts
that are at best tangentially relevant to the others.

**How to apply:** when routing a fact up, strip the `[[...]]` and keep the
prose that names the incident — "confirmed twice on ghmem's `sync.py`" is
evidence and belongs in the tier; the pointer is the only part that crossed the
boundary. Where the link *was* the whole reference ("Related:
[[some-project]]"), delete it; a tier fact stands on its own Why and How-to-
apply or it is not a tier fact. Watch for two adjacent cases: a link naming a
*skill* rather than a memory, and one whose target the prose already names, so
the pointer is pure redundancy.

**A repo-relative file path dies the same way, and more quietly.** A tier fact
citing `docs/references/cc-memory-retrieval.md` resolves only inside the repo
that holds the file; from every other consumer it names nothing, and the reader
cannot tell which repo owns it. Name the repo in the prose — "the gitlore
repo's `docs/references/…`" — or drop the citation. Observed on
[[index-compaction-triggers]], whose pointer to the recall research read from
micro as a missing file; the research was re-run from scratch and reproduced
findings already recorded there. Where the citation is the record of prior
work, name the artifacts too, so the next session can see the question is
already answered before spending a session answering it again.

See [[memory-writing]] and [[gitlore-tier-index-budget]].
