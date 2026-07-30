---
name: guide-agent-sonnet-minimum
description: Run claude-code-guide agents on Sonnet at least — Haiku is unreliable in that role
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 2eeaac7d-8f74-4a8c-941d-66cd37826d27
---

When consulting the claude-code-guide agent (or any "ask the docs"
research role), always set the model to Sonnet or better.

**Why:** my human partner found Haiku unreliable for tricky guide
tasks — it misses useful information and occasionally hallucinates.

**How to apply:** pass `model: sonnet` (or higher) on every
claude-code-guide Agent call; never let a guide query default to a
smaller model.
