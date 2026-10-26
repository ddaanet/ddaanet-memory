---
name: owned-projects
description: "inventory of the projects my human partner owns and an agent may route a fix to — plugin, skills, agents, hooks, inbox per repo under ~/code; read by name at the memory gate, not indexed"
metadata:
  node_type: memory
  type: reference
  originSessionId: c7552bfd-42e8-494d-9267-e79406b2fbd0
  modified: 2026-10-26T17:40:19.156Z
---

Projects my human partner owns, by checkout under `~/code/`. A fact that one of
these owns — a skill body, an agent definition, a hook's message, a tool's
output — goes there, not into memory: fixed at the source when the owner is
this repo, and as a brief in the owner's `inbox/` when it is a sibling (other
repos stay read-only beyond that brief). Every repo here is modifiable by its
own session. A project not listed is asked about at the memory gate, and the
answer is recorded here.

Read by name from each mounted tier's `owned-projects.md`; it has no index
line. Visibility is read from the remote; a private repo's content stays out
of anything committed to a public one.

| Checkout | Plugin | Ships | inbox/ |
|---|---|---|---|
| gitlore | gitlore | skills memory-writing, merge, push, recall, resolve; agent memory-merger; hooks | yes |
| handoff | autodrive-handoff | skills autoname, handoff, pending, precompact; hooks | yes |
| craft | craft | skills design-doc-writing, directive-writing, plan-contracts, removing-cleanly, test-discipline | yes |
| plugin-craft | plugin-craft | skills hook-authoring, skill-authoring, toolkit-release, verifying-plugin-changes | yes |
| sandbox-lies | sandbox-lies | skill sandbox-lies; hooks | yes |
| shell-gotchas | shell-scripting | skill shell-gotchas; hooks | yes |
| ghmem | ghmem | agents ghmem-synthesize, ghmem-triage; hooks | yes |
| gitmoji | gitmoji | hooks (commit-msg) | yes |
| onekeys | onekeys | hooks | yes |
| cwd-safety | cwd-safety | hooks | yes |
| prohibitions | prohibitions | hooks | yes |
| candidature | candidature | skill candidature; hooks | yes |
| claude-plugin-dev | — | the vendored `plugin-dev/` toolkit (gates, run-bats, release) | yes |
| claude-plugins | — | the `ddaanet` marketplace | no |
| edify | — | Claude Code toolkit: Python CLI and skills | yes |
| micro-craft | — | design knowledge about the micro editor | yes |
| pytest-md | — | pytest markdown report plugin | no |
| micro-contrib | — | private | no |
| Emploi | — | private | no |

The memory tier itself is `ddaanet/ddaanet-memory`, mounted as `ddaanet/`.
