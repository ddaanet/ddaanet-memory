---
name: plugin-agent-hooks-ignored
description: "a plugin-shipped agent (`<plugin>:<agent>`) runs uncontained: `sets hooks, which is ignored for plugin agents` (debug log only); contain via plugin `hooks/hooks.json` on `agent_type`"
metadata:
  node_type: memory
  type: reference
  originSessionId: 86dd0f9f-ce94-4f31-bf3a-2ff5f119ad1a
  modified: 2026-10-13T11:13:54.957Z
---

An agent definition shipped under a plugin's `agents/` directory gets none
of its frontmatter `hooks:`. The loader drops them when it reads the file
and says so only at WARN on the debug channel (`claude --debug`):

```
Plugin agent file <path>/agents/<name>.md sets hooks, which is ignored
for plugin agents. Use .claude/agents/ for this level of control.
```

Verified on CC 2.1.296 with the plugin loaded through
`CLAUDE_CODE_PLUGIN_DIRS`: a tripwire dispatch of the agent ran a command
its `PreToolUse` hook exists to block, and the command executed.

Nothing else reports it. The dispatch looks correct from every other
angle — the subagent's `.meta.json` shows `agentType: "<plugin>:<agent>"`
with no `customAgentType`, the agent is found and runs — so the checks
that catch the two `.claude/agents/` gates in
[[frontmatter-hooks-two-silent-gates]] all pass. The plugin source being
exempt from that file's workspace-trust check is a separate matter: the
exemption never gets a chance to apply.

**How to apply:** an agent whose safety depends on a hook cannot get it
from its own frontmatter once it ships in a plugin. The two routes left:

- a plugin-level `hooks/hooks.json` whose scripts read the payload's
  `agent_type` and act only for the plugin's own agents. `agent_type` is
  present on hook payloads fired from inside a subagent; the exact value
  for a plugin agent is worth one probe before relying on it.
- definitions installed into the consumer project's `.claude/agents/`,
  where frontmatter hooks do attach — subject to both gates in
  [[frontmatter-hooks-two-silent-gates]].

A test that the frontmatter's hook paths resolve proves nothing about
containment for a plugin agent. Verify with a tripwire dispatch and read
the tool result: one command that must be blocked, one that must be
allowed. If the session's permission rules would deny the tripwire
anyway, allow it there first — a permission denial is indistinguishable
from a hook block in the agent's report.
