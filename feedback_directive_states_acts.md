---
name: feedback-directive-states-acts
description: "An agent-facing directive states only the acts its reader performs — no mechanism, no restating what earlier steps already settled"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 8cd21851-d241-49bf-afaf-b4baf0c4f311
  modified: 2026-07-25T19:03:45.029Z
---

A directive emitted to an agent (hook output, probe stdout, skill step) carries
only the acts that reader performs. Three things do not belong in it:

- **Mechanism.** How the machinery consumes the act — a pre-commit hook, an
  mtime freshness rule, a gitlink being staged.
- **What an earlier step already settled.** If the skill body sequences memory
  before the probe, the probe must not re-order them.
- **A meta-description of the constraint.** "no second file, no trigger" was a
  note about what the text must *avoid*; it got literalized into the text.

**Why:** mechanism a reader cannot act on does not sit inert — it gets
verified, narrated back to the user, and worked around. Guidance restating a
condition the reader has already satisfied asks a question whose only possible
answer is narration. Both show up as an agent reporting superfluous checks.

**How to apply:** write the acts, then delete every sentence that explains why
they work. Keep a consequence clause only where its absence would read as a
failure worth recovering from ("this turn's commit carries the memory"). When a
rule is an ordering between the *caller's own steps*, it belongs where the steps
are described — see [[feedback_prohibition_vs_procedure]] on placement, and
[[feedback_withhold_dont_forbid]] for the case where the right amount of text is
none.
