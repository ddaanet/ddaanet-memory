---
name: named-dispatch-drops-frontmatter-hooks
description: "spawning a custom agent with a `name` overwrites the dispatch's `agentType` and demotes the definition to `customAgentType`; frontmatter `hooks:` resolve by `agentType`, so a named dispatch registers zero hooks and runs uncontained"
metadata: 
  node_type: memory
  type: reference
  originSessionId: ca597732-d159-40c8-adc3-c5e8c88c1ec2
  modified: 2026-07-30T15:34:22.776Z
---

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
worker mid-flight, which batch workers never need. Verify containment
by dispatch-shape, not by asking: check `agentType` in the
`.meta.json`, or have the definition open with a tripwire — one
command that *must* be blocked, with instructions to stop and report
UNCONTAINED if it instead succeeds.

Beware the corroboration this hides in plain sight: a project's whole
dispatch history can look uniform ("every custom-agent dispatch shares
this shape") while the one differing field is the one that matters.
Related: [[agent-hooks-need-exact-trust-key]] (a *separate* gate, also
silent) and [[transcripts-are-ground-truth]].
