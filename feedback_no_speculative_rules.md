---
name: feedback_no_speculative_rules
description: "an anti-pattern records a defect that happened; no incident, no entry; state a needed rule positively in the step where it acts"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 8cd21851-d241-49bf-afaf-b4baf0c4f311
  modified: 2026-07-26T14:00:09.917Z
---

Do not add a rule or anti-pattern to a skill body for a failure nobody has seen. An anti-pattern list is a record of defects that happened; padding it with plausible-sounding ones dilutes the real entries and costs context on every load.

A second, sharper failure mode has been seen once, and is recorded as one instance rather than a rate: an invented guardrail around a new decision point was phrased as a prohibition on the agent, and the request that reaches that decision point was the one that wanted the forbidden action. Whether that generalizes is unknown — but it is cheap to avoid, because the fix is the same one the primary rule already implies.

**Why:** when handoff gained a `with-commit`/`without-commit` step, three entries went in unprompted — "defaulting the answer instead of deciding it", "running git because the answer was `with-commit`", and a declarative "this skill never commits". User: "those two anti patterns are speculative. besides, when the user says 'handoff and commit' (or some variation, the common expected case) forbidding git commands prevents the expected behaviour." All three came out; the one ordering constraint that mattered was already stated positively in the step it applies to.

**How to apply:** before adding an anti-pattern, name the incident. No incident, no entry. When a rule must exist, state it positively in the step where it acts ("the commit lands before this file is written") rather than as a prohibition on the agent — a positive ordering constrains sequence without forbidding the action. Related: [[feedback_directive_states_acts]] (same economy applied to script-emitted directives), [[feedback_remove_cleanly_no_vestigial]].
