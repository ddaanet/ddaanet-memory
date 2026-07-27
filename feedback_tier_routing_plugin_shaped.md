---
name: feedback_tier_routing_plugin_shaped
description: "litmus test for routing an operational/environmental gotcha to the ddaanet tier vs project memory: would it affect anything plugin-shaped, not just this repo"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a3e2f110-a3fe-4f99-bfb6-a2343f2a4110
  modified: 2026-07-27T21:44:28.305Z
---

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
