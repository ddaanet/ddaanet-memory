---
name: vibe-hooks-agents-spawn
description: "Vibe dispatch is `agents.spawn`: hooks observe it via `tool_input` `{name,message}`, rewrites are inert, deny works"
metadata:
  node_type: memory
  type: reference
  originSessionId: f16a6bed-a953-4e46-94d8-6ea12bb015d3
  modified: 2026-11-28T14:26:49.352Z
---

Pinned 2026-11-25 against the installed mistral-vibe source
(`vibe/core/plugins/_compatibility.py`, `_native.py`, the
`mistralai_vibe_local_harness` hooks runner) and a
`claude plugin validate --strict` probe.

## Dispatches: hooks see agents.spawn, rewrites are inert on it

Probed 2026-11-26 (Vibe 3.0.1, headless sessions with catch-all and
spawn-targeted hooks). There is no `task`-named tool call in a Vibe session:
a dispatch travels as a programmatic execution of `tools.agents.spawn`
inside `run_typescript` (session record: `toolName: "agents.spawn"`, input
normalized `{agent, task}`). The authoritative matcher names, from
`mistralai_vibe_local_harness/vibe/_hook_matcher.py` `candidate_names`, are
`agents.spawn`, `agents_spawn`, `spawn` (plus `re:`/fnmatch recipes) —
matching is at call time, back-compat legacy names included.

- Hooks DO observe spawns: `pre_tool` and `post_tool` fire with
  `tool_name = "agents.spawn"`, `tool_input = {name, message}` where
  `message` is the child's admitted first message. A denied call fires no
  `post_tool`.
- **Rewrites are inert on the programmatic path**: a hook's
  `hook_specific_output.tool_input` rewrite of `message` reached Core (the
  rewrite was logged) but the child received the original text — the
  programmatic external execution runs the requested arguments, not the
  rewritten ones. Injection into a dispatch prompt is therefore impossible
  via hooks; the only hook-expressible act on a spawn is **deny**, which
  works: no child session is created and the reason reaches the model as
  the tool's `tool_skipped` error while the session continues.
- The child is its own session (reads the repo `AGENTS.md` itself, carries
  `hook_bindings []`), which is why a rewrite would not matter for the
  AGENTS.md-visible parent in-framework, and why nothing injects into a
  subagent in a repo whose AGENTS.md carries no store line.
