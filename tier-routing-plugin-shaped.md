---
name: tier-routing-plugin-shaped
description: "routing an operational gotcha to ddaanet vs project memory"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a3e2f110-a3fe-4f99-bfb6-a2343f2a4110
  modified: 2026-08-10T19:00:51.923Z
---

When a fact looks project-local at first glance — a sandbox quirk, an
environment gotcha, a harness interaction — check whether it's actually
scoped to *this repo's content* or to *the class of repo this is* (a
Claude Code plugin, a git-hooked project, etc.) before filing it under
project memory.

**Why:** a mechanism that fires for a whole class of repo has as many
consumers as the class has members, and filing it project-local hides it
from all but one. Evidence naming the repo where it surfaced is not
evidence the mechanism depends on that repo — a sandbox EROFS on a
plugin's own active `hooks/hooks.json` reads as repo-specific, but the
harness mounts any live plugin config read-only, so it fires for every
plugin-shaped repo ([[sandbox-effects]]).

**How to apply:** ask "does this fact's *mechanism* depend on something
unique to this repo, or just on this repo being a member of a class
(plugin, hook-managed repo, submodule-based memory store, …)?" If it's the
latter, route to `memory/ddaanet/` (or the matching tier), merged into an
existing topic file if one already exists for that trigger surface —
don't leave a same-fact stub in the project tier "just in case."
