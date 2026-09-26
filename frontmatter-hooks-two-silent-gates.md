---
name: frontmatter-hooks-two-silent-gates
description: "two independent gates drop a subagent's `.claude/agents` frontmatter `hooks:` with no error and no UI signal (\"Skipping frontmatter hooks\", `tengu_agent_hooks_origin_untrusted`): `~/.claude.json` `hasTrustDialogAccepted` must name the exact project path (inherited parent trust hides it, no dialog ever appears), and a dispatch `name` overwrites `agentType` with `customAgentType` so zero hooks register and the agent runs uncontained — check `agentType` in the subagent `.meta.json`, never ask the agent"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 0e54e20b-7997-422c-b864-72737115682c
  modified: 2026-09-26T15:17:00.491Z
---

A project subagent's frontmatter `hooks:` can be dropped by **two independent
gates**, each silent. Either one closed is on its own enough to drop the hooks;
both must be clear before they attach, so clearing one alone restores nothing —
writing the trust key changed nothing here, because every uncontained dispatch
had also been spawned under a custom `name`.

Which one, by what you can observe:

- Hooks inert for an agent defined in **this repo's `.claude/agents/`**, no
  trust dialog ever offered, `tengu_agent_hooks_origin_untrusted` in telemetry
  or `Skipping frontmatter hooks for agent '…'` in the debug log →
  `memory/ddaanet/frontmatter-hooks-trust-key-exact.md`.
- Hooks inert for a dispatch given a custom `name`, `agentType` in the
  subagent's `.meta.json` holding the name rather than the definition →
  `memory/ddaanet/frontmatter-hooks-dispatch-name-demotes.md`.

**Verify by dispatch shape, never by asking.** An agent cannot observe a hook
that was never attached, so it reasons from its definition and reports itself
contained. Read `agentType` in the `.meta.json` instead. With the trust key set
*and* the dispatch unnamed, hooks attach and block correctly (verified
2026-07-25 at the tool result). A project's whole dispatch history can look
uniform — "every custom-agent dispatch shares this shape" — while the one
differing field is the one that matters.

Related: [[verify-restart-before-structural-diagnosis]] and
[[transcripts-are-ground-truth]].
