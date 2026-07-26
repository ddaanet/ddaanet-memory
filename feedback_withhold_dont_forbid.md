---
name: feedback-withhold-dont-forbid
description: "naming an identifier in order to prohibit it supplies the string; ask who else could supply it, and when nobody can, say nothing instead of forbidding"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 8cd21851-d241-49bf-afaf-b4baf0c4f311
  modified: 2026-07-26T12:06:15.060Z
---

When the only route to an unwanted action is an identifier — a filename, a
flag, an endpoint — that the reader has no other source for, withhold it.
Do not name it in order to forbid it.

**Why:** the prohibition introduces the concept, and an introduced concept gets
reasoned about. Naming `.claude/gitlore-commit-memory` to say "do not write it"
hands a fresh agent the exact string that triggers the behaviour the mode exists
to prevent. Withholding is not politeness — it is what makes the action
unreachable. The counter-argument ("a reader who already knows the protocol
needs it forbidden, not omitted") assumes a rare reader and pays for it with the
common one; maintainer docs like a project `CLAUDE.md` are not in a foreign
project's context.

**How to apply:** ask who else could supply the identifier. Nobody → say
nothing. Only when the reader demonstrably already has it (it appeared earlier
in the same context) is a prohibition worth its cost, and then forbid by role
rather than by name. Test it as a negative on the whole word, and mutation-check
it per [[feedback_mutation_check_negatives]]. Related:
[[feedback_directive_states_acts]], and
[[feedback_directive_acts_not_mechanism]], which carries this as one bullet
inside the wider directive-economy rule — this entry is the standalone test and
the counter-argument.
