---
name: feedback_directive_acts_not_mechanism
description: "script-emitted text an agent acts on carries only the acts; mechanism gets checked and narrated back, and \"do NOT write `<path>`\" hands over the string it forbids — silence guards better when the reader has no other source"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 8cd21851-d241-49bf-afaf-b4baf0c4f311
  modified: 2026-07-26T12:06:32.587Z
---

When authoring text a script emits *for an agent to act on* (a probe directive, a hook's `additionalContext`, an error message that instructs), include only the acts the reader performs. Cut everything else:

- **Mechanism** — what the receiving system does with the file, which hook consumes it, the freshness rule behind an ordering. The reader cannot act on it, so it becomes checks it runs, prose it narrates back to the user, and constraints it tries to work around. State the ordering; never the rule that motivates it.
- **Prohibitions naming a thing the reader could not otherwise reach.** "Do NOT write `<path>`" hands over the one string needed to do it. Silence is the stronger guard when the reader has no other source for the name — and a prohibition also *introduces the concept*, which then gets reasoned about.
- **Rationale for why the mode exists.** The reader is executing, not reviewing the design.
- **What an earlier step already settled.** If the skill body sequences memory before the probe, the probe must not re-order them — restating a condition the reader has already satisfied asks a question whose only answer is narration.
- **A meta-description of the constraint.** "no second file, no trigger" was a note about what the text must *avoid*; it got literalized into the text itself.

Keep one clause of reassurance when the absence of a visible result would otherwise read as failure ("the commit that lands these changes carries the memory") — that prevents an invented recovery. Make it true of every flow the mode covers: the first draft said "this turn's commit", which is false wherever the commit arrives in a later turn — see [[feedback_turn_call_terminology]].

**Why:** handoff's `with-commit` memory directive shipped as five paragraphs explaining gitlore's pre-commit hook, gitlink staging, mtime freshness, and an explicit "do NOT write `.claude/gitlore-commit-memory`". User: "This is still saying much more than the agent needs to know. That kind of prose leads to annoying verbosity when the agent narrates the superfluous checks." On the prohibition: the reader is a fresh agent in an arbitrary project with no other source for that filename, so naming it — even to forbid it — was the only way it could cause the split the mode exists to prevent.

**How to apply:** draft the directive, then delete every sentence that is not an act. Ask what a reader would *do* with each remaining line; if the answer is "check it" or "explain it back", cut it. Test the negatives (`[[ "$output" != *"trigger"* ]]`) and mutation-check them per [[feedback_mutation_check_negatives]]. Same timing logic as [[feedback_prohibition_vs_procedure]] applied to content rather than placement; the split of judgement from mechanism is [[feedback_harness_over_agent]]. Two narrower entries from the same review cover parts of this in their own right: [[feedback_directive_states_acts]] is the bare taxonomy of what to cut, and [[feedback_withhold_dont_forbid]] is the withhold-the-identifier rule with its own test and counter-argument.
