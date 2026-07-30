---
name: model-tiered-workflow
description: "my human partner runs a cost-tiered pipeline — Fable designs, Opus drafts plans, Sonnet executes; Fable output stays short, prose-only, no code chunks"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a51ff8d4-df99-4c6a-be00-e4748ad9ef49
  modified: 2026-07-30T16:00:52.247Z
---

My human partner is cost-conscious about Fable. He wants Fable for high-level design and review
only: short, to the point, prose outlines, explicitly "don't generate chunks of code,
cheaper agents do that fine from a prose outline". Plan drafting goes to an Opus
subagent (Fable reviews the draft); execution goes to Sonnet in a later session. Use
subagents proactively for exploration and bulk writing.

**Why:** Fable tokens are expensive; a pinned-down spec makes drafting mechanical, and
plan review costs a fraction of plan generation.

**How to apply:** On any multi-stage task, keep Fable turns lean (design decisions,
verdicts, fixes), delegate file-producing work to Opus/Sonnet subagents with
self-contained prompts, and review their output against the spec instead of redoing it.
Follow the project's own convention for where specs/plans live.
