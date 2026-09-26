---
name: frontmatter-hooks-trust-key-exact
description: "gate 1 of [[frontmatter-hooks-two-silent-gates]]: the frontmatter-hook trust check is an exact-key lookup on `~/.claude.json` `hasTrustDialogAccepted`, so a repo trusted through an ancestor never gets a dialog and its agents' hooks are dropped"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 2f7bce41-1003-4688-8245-cf58734a7bc4
  modified: 2026-09-26T15:15:31.846Z
---

Gate 1 of [[frontmatter-hooks-two-silent-gates]]. Claude Code has **two
different workspace-trust checks**, and they disagree. A repo can be trusted for
every ordinary purpose while a subagent's frontmatter `hooks:` are silently
dropped.

The session-level check walks **up** the directory tree: if any ancestor has
`hasTrustDialogAccepted: true` in `~/.claude.json`, the workspace is trusted,
the trust dialog never appears, and the per-project flag is never written. A
parent directory holding every checkout is such an ancestor, so each repo under
it inherits trust.

The agent-frontmatter-hook check is an **exact-key lookup with no parent walk
and no sandbox or bypass escape hatch**. The key is the folder the definition
file came from, with `/.claude/agents` stripped — i.e. the project root. Sources
`plugin`, `policySettings`, `built-in`, `bundled`, `userSettings` and
`flagSettings` are exempt; a project's `.claude/agents/` is not.

So an inherited-trust repo fails the strict check, and the failure is invisible:
an error-level line in the debug log (`Skipping frontmatter hooks for agent
'…': the folder its definition file came from is not trusted`) plus telemetry
`tengu_agent_hooks_origin_untrusted`, with nothing in the UI. The agent runs
with none of its declared containment. The telemetry event is still in the CC
2.1.283 binary, which places the gate but says nothing about its wording.

**How to apply:** when a project subagent's hooks appear inert, do not conclude
the hook scripts or agent files are wrong, and do not go looking for a trust
dialog to accept — under an already-trusted parent there is none to find. Check
`projects["<exact project path>"].hasTrustDialogAccepted` in `~/.claude.json`
and write it `true` (the product's own error message sanctions this), then
restart. Alternatively move the definitions to `~/.claude/agents/` or ship them
as a plugin, both exempt sources, or register the hook in
`.claude/settings.json`, where no trust check applies.
