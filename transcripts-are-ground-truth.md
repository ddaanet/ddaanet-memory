---
name: transcripts-are-ground-truth
description: "a corrupted subagent artifact: transcripts hold the verbatim tool inputs to diff against"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: ca597732-d159-40c8-adc3-c5e8c88c1ec2
  modified: 2026-08-01T14:37:03.486Z
---

When an agent's output lands corrupted (shell expansion, truncation, a
partial write), do not try to judge the artifact by inspection. The
subagent transcript records the verbatim tool input, so the intended
text is recoverable and the artifact can be diffed against it
character by character.

Named-subagent transcripts live at
`~/.claude/projects/<slug>/<parent-session-id>/subagents/agent-*.jsonl`;
each `tool_use` block carries the unexpanded `input.command`.

**Why:** the ghmem routing-ledger contamination (backticks executed
inside double-quoted `ghmem route` notes) was written up as
unverifiable — corrupt, repaired and clean notes all read as plausible.
Diffing 27 ledger notes against their issuing commands settled every
one of them: 24 byte-identical, one lossy-truncated, one silently
gutted, one hand-repaired and provably faithful. What "cannot be
verified" often just has an unread source of truth.

**How to apply:** before proposing re-doing work because its output
cannot be trusted, check whether a transcript pins down what was meant.
Parse the command with `shlex.split` — it keeps backticks literal, so
what it yields is the author's intent, and any difference from the
artifact is exactly what the shell did to it. Relates to
[[verify-restart-before-structural-diagnosis]].
