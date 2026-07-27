---
name: reference-skill-bundled-scripts
description: "CLAUDE_PLUGIN_ROOT is UNSET in agent Bash (hooks only); every *enabled* plugin's bin/ is on PATH (cross-plugin bare-name invocation works; disabled = absent), so invoke bundled scripts by a bare-name `bin/` shim, no hook needed"
metadata:
  node_type: memory
  type: reference
  originSessionId: 2374725e-5904-4dcf-b8f6-67e3f33aa929
  modified: 2026-07-24T19:24:30.921Z
---

How a SKILL.md body reliably invokes a script bundled in its own plugin —
**verified by test 2026-06-12** (CC 2.1.175) and re-verified by the same live
`env`/`PATH` probe on **CC 2.1.219, 2026-07-24**, contradicting two
claude-code-guide passes that claimed it isn't possible:

- `CLAUDE_PLUGIN_ROOT` is **UNSET** in the agent's Bash tool environment (it is
  set only for hook processes). So `${CLAUDE_PLUGIN_ROOT}/scripts/foo.sh` from a
  skill body does NOT resolve. (`env` dump confirmed; `CLAUDE_PROJECT_DIR` is
  absent from agent Bash too.)
- Claude Code **auto-adds every installed plugin's `<root>/bin` to PATH** — no
  `bin` key in `plugin.json` needed (the cache plugin bins are all on PATH).
  This holds from the cache path, so it works for end-users, not just a dev
  checkout.
- **Cross-plugin bare-name invocation works** (verified 2026-07-12): PATH gets
  one `bin/` entry per *enabled* plugin — all plugins, not just the invoking
  one, and entries are added even when the `bin/` dir doesn't exist (the PATH
  builder is `enabled.filter(non-builtin && path).map(join(path,"bin"))`, with
  no existence check). Disabled plugins' bins are absent. So plugin A's skill
  can call plugin B's `bin/` executable by bare name iff B is enabled;
  `command -v` is the availability probe (costs a tool call — see the handoff
  single-turn FR before using it).
- A `bin/` executable **resolves by bare name** from a fresh agent Bash call
  (tested: a throwaway `bin/handoff-probe-test` ran by name → printed output).
- The Skill tool also announces `Base directory for this skill: <abs-path>` at
  invocation; Anthropic's document-skills (xlsx/pptx/docx) lean on this with
  bare relative `python scripts/foo.py`. Less reliable across invocation paths
  than the `bin/`-on-PATH route. Re-confirmed 2026-07-22 on a **slash-command**
  invocation of a plugin skill, which is the path most likely to differ; the
  announce string is still in the 2.1.219 bundle.
- Consequence for **cross-skill references**, distinct from the bundled-script
  case: sibling skills in one plugin are reachable from that base directory by
  relative path (`../other-skill/SKILL.md`). A skill that points at another
  skill's template should name that path. Pointing at "the template in the X
  skill" with no path leaves *invoking X* as the only route the agent can see —
  and a skill with an activation hook does destructive work on invocation.

**Canonical pattern:** put a uniquely-named thin shim in the plugin's `bin/`
(`exec bash "$(dirname "$0")/../scripts/real.sh" "$@"`), keep logic in
`scripts/`, and have the skill body invoke the shim by bare name. No hook
required for "skill body must run a bundled script." Use a hook only when no
agent action is needed on the result. See [[feedback-harness-over-agent]].
