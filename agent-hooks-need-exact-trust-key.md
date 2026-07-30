---
name: agent-hooks-need-exact-trust-key
description: "`.claude/agents` frontmatter `hooks:` silently skipped (\"Skipping frontmatter hooks\", `tengu_agent_hooks_origin_untrusted`) when `~/.claude.json` `hasTrustDialogAccepted` is false for the exact project path; inherited parent trust hides it and no dialog ever appears — necessary but not sufficient, a second silent gate is the dispatch's name, and a provoke-check must read the tool result, not the agent's report"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 0e54e20b-7997-422c-b864-72737115682c
  modified: 2026-07-30T15:34:16.627Z
---

Claude Code has **two different workspace-trust checks**, and they
disagree. A repo can be trusted for every ordinary purpose while a
subagent's frontmatter `hooks:` are silently dropped.

The session-level check walks **up** the directory tree: if any ancestor
has `hasTrustDialogAccepted: true` in `~/.claude.json`, the workspace is
trusted, the trust dialog never appears, and the per-project flag is
never written. `/Users/david/code` is such an ancestor, so every repo
under it inherits trust.

The agent-frontmatter-hook check is an **exact-key lookup with no parent
walk and no sandbox or bypass escape hatch**. The key is the folder the
definition file came from, with `/.claude/agents` stripped — i.e. the
project root. Sources `plugin`, `policySettings`, `built-in`, `bundled`,
`userSettings` and `flagSettings` are exempt; a project's
`.claude/agents/` is not.

So an inherited-trust repo fails the strict check, and the failure is
invisible: an error-level line in the debug log
(`Skipping frontmatter hooks for agent '…': the folder its definition
file came from is not trusted`) plus telemetry
`tengu_agent_hooks_origin_untrusted`, with nothing in the UI. The agent
runs with none of its declared containment.

**Writing the key did not by itself make the hooks fire**, because a
second, unrelated gate was also closed: every dispatch that ran
uncontained had been spawned under a custom `name`, which drops
frontmatter hooks outright — see
[[named-dispatch-drops-frontmatter-hooks]]. With the trust key set
*and* the dispatch unnamed, hooks attach and block correctly
(verified 2026-07-25 at the tool result). Treat the trust key as
necessary, not sufficient, and check the dispatch shape before
suspecting anything structural.

**How to apply:** when a project subagent's hooks appear inert, do not
conclude the hook scripts or agent files are wrong, and do not go
looking for a trust dialog to accept — under an already-trusted parent
there is none to find. Check
`projects["<exact project path>"].hasTrustDialogAccepted` in
`~/.claude.json` and write it `true` (the product's own error message
sanctions this), then restart. Alternatively move the definitions to
`~/.claude/agents/` or ship them as a plugin, both exempt sources, or
register the hook in `.claude/settings.json` where no trust check
applies.

Verify with a provoke-and-check dispatch whose probe is harmless if the
hooks are still dead — an `echo` and a throwaway file, never real data.
**Read the probe's tool result, not the agent's report of it.** The
2026-07-24 check here was recorded as proving containment live; its
transcript shows `echo hello && whoami` returning `hello / david`. The
agent said contained, the tool result said executed, and the agent was
believed. See [[verify-restart-before-structural-diagnosis]] and
[[guardrail-must-permit-real-commands]].
