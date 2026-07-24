---
name: feedback-prose-over-modals-design
description: "AskUserQuestion modals fine ~75% of the time, but never foreclose prose for clarification, correcting assumptions, or adjusting; ask in prose when proposing architecture"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 2374725e-5904-4dcf-b8f6-67e3f33aa929
---

`AskUserQuestion` modals are fine roughly 75% of the time. The failure is
forcing *everything* through them: prose is often needed for clarification,
for correcting a wrong assumption baked into the options, or for adjusting the
response. In the gitlore-aware design session the user interrupted three
successive modals, each time answering in prose and reframing the architecture
("proactive" → "we need a script entry point to supplement the pre-commit
hook") — the fixed options presumed the wrong frame.

**Why:** a modal presents a closed option set; early architecture exploration
and assumption-correction are open, so a modal there forces an interruption to
reframe.

**How to apply:** keep using modals for genuinely closed choices, but don't let
one foreclose prose. When proposing an architecture direction, or when the user
may need to correct an assumption, ask in prose (or make prose the easy path).
