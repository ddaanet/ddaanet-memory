---
name: frontmatter-hooks-two-silent-gates
description: "two independent gates drop a subagent's `.claude/agents` frontmatter `hooks:` with no error and no UI signal (\"Skipping frontmatter hooks\", `tengu_agent_hooks_origin_untrusted`): `~/.claude.json` `hasTrustDialogAccepted` must name the exact project path (inherited parent trust hides it, no dialog ever appears), and a dispatch `name` overwrites `agentType` with `customAgentType` so zero hooks register and the agent runs uncontained — check `agentType` in the subagent `.meta.json`, never ask the agent"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 0e54e20b-7997-422c-b864-72737115682c
  modified: 2026-10-13T11:13:05.065Z
---

A project subagent's frontmatter `hooks:` can be dropped by **two
independent gates**, each silent. Either one closed is on its own enough to
drop the hooks; both must be clear before they attach, so clearing one alone
does not restore containment.

Merged from two facts recorded in sessions `0e54e20b-7997-422c-b864-72737115682c`
(trust key) and `ca597732-d159-40c8-adc3-c5e8c88c1ec2` (named dispatch).

## Gate 1 — the workspace-trust key must be exact

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

**How to apply:** when a project subagent's hooks appear inert, do not
conclude the hook scripts or agent files are wrong, and do not go
looking for a trust dialog to accept — under an already-trusted parent
there is none to find. Check
`projects["<exact project path>"].hasTrustDialogAccepted` in
`~/.claude.json` and write it `true` (the product's own error message
sanctions this), then restart. Alternatively move the definitions to
`~/.claude/agents/`, an exempt source, or register the hook in
`.claude/settings.json` where no trust check applies. Shipping them as
a plugin is not an alternative: see [[plugin-agent-hooks-ignored]].

## Gate 2 — a dispatch `name` demotes the agent type

Passing a custom `name` to the Agent tool **silently disables an agent
definition's frontmatter `hooks:`**.

The dispatch metadata (`~/.claude/projects/<slug>/<session>/subagents/
agent-*.meta.json`) shows the mechanism. Named:

```
"agentType": "triage-1",  "customAgentType": "ghmem-triage"
```

Unnamed:

```
"agentType": "ghmem-triage",  "customAgentType": null
```

The name overwrites `agentType` and demotes the real definition to
`customAgentType`. Frontmatter hooks are registered by resolving
`agentType`, so a named dispatch resolves an agent that does not
exist, registers zero hooks, and runs with whatever unrestricted tool
access its `tools:` list allows.

**Why it stays invisible:** nothing reports it in either direction. No
error, no UI signal, and the agent itself cannot tell — it has no way
to observe a hook that was never attached, so an agent asked "were you
contained?" will reason from its definition and answer yes. The
harness logs registration only on the debug channel (`Registered N
frontmatter hook(s) from <source> for session <id>`), which is off by
default.

**How to apply:** spawn agents that carry frontmatter hooks by
`subagent_type` alone. The only thing a name buys is addressing the
worker mid-flight, which batch workers never need.

## Verifying, once both gates are clear

Writing the trust key did not by itself make the hooks fire, because
every dispatch that ran uncontained had also been spawned under a
custom `name`. With the trust key set *and* the dispatch unnamed, hooks
attach and block correctly (verified 2026-07-25 at the tool result).
Treat each gate as necessary, not sufficient, and check the dispatch
shape before suspecting anything structural.

Verify containment by dispatch-shape, not by asking: check `agentType`
in the `.meta.json`, or have the definition open with a tripwire — one
command that *must* be blocked, with instructions to stop and report
UNCONTAINED if it instead succeeds. Use a provoke-and-check dispatch
whose probe is harmless if the hooks are still dead — an `echo` and a
throwaway file, never real data. **Read the probe's tool result, not
the agent's report of it.** The 2026-07-24 check here was recorded as
proving containment live; its transcript shows `echo hello && whoami`
returning `hello / david`. The agent said contained, the tool result
said executed, and the agent was believed. Pair the tripwire with a
command that must be ALLOWED in the same dispatch — a guard broken
*closed* blocks the provocation too and looks exactly like success from
the deny alone.

Beware the corroboration this hides in plain sight: a project's whole
dispatch history can look uniform ("every custom-agent dispatch shares
this shape") while the one differing field is the one that matters.

Related: [[verify-restart-before-structural-diagnosis]] and
[[transcripts-are-ground-truth]].
