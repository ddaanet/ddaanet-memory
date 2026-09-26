---
name: frontmatter-hooks-dispatch-name-demotes
description: "gate 2 of [[frontmatter-hooks-two-silent-gates]]: a custom `name` on the Agent tool overwrites `agentType` with `customAgentType`, so frontmatter `hooks:` resolve against an agent that does not exist and zero register"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 2f7bce41-1003-4688-8245-cf58734a7bc4
  modified: 2026-09-26T15:15:54.399Z
---

Gate 2 of [[frontmatter-hooks-two-silent-gates]]. Passing a custom `name` to the
Agent tool **silently disables an agent definition's frontmatter `hooks:`**.

The dispatch metadata
(`~/.claude/projects/<slug>/<session>/subagents/agent-*.meta.json`) shows the
mechanism. Named:

```
"agentType": "triage-1",  "customAgentType": "ghmem-triage"
```

Unnamed:

```
"agentType": "ghmem-triage",  "customAgentType": null
```

The name overwrites `agentType` and demotes the real definition to
`customAgentType`. Frontmatter hooks are registered by resolving `agentType`, so
a named dispatch resolves an agent that does not exist, registers zero hooks,
and runs with whatever unrestricted tool access its `tools:` list allows.

**Why it stays invisible:** nothing reports it in either direction. No error, no
UI signal, and the agent itself cannot tell — it has no way to observe a hook
that was never attached, so an agent asked "were you contained?" reasons from
its definition and answers yes. The harness logs registration only on the debug
channel (`Registered N frontmatter hook(s) from <source> for session <id>`),
which is off by default.

**How to apply:** spawn agents that carry frontmatter hooks by `subagent_type`
alone. The only thing a name buys is addressing the worker mid-flight, which
batch workers never need.
