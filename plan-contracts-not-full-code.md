---
name: plan-contracts-not-full-code
description: "how much code a plan carries; writing-plans demands full bodies and must be overridden"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a9133ea6-55ff-41b0-a76f-21cad29dbb69
  modified: 2026-08-01T14:28:28.871Z
---

Implementation plans are **contract-style**. Verbatim code belongs in a
plan only where a mismatch across a task boundary would be a real
defect; everything else is stated as a contract and the implementer
writes the code.

- **Verbatim, always:** signatures, types, names, and literal strings
  that cross a task or process boundary — a `Literal[...]` union's
  members, a header prefix an agent parses, a terminal message. A
  reviewer reading one task cannot catch a mismatch here.
- **Contract, never code:** function bodies. State the precondition,
  the postcondition, and the invariant.
- **Arguably verbatim:** tests, since a test is the contract made
  executable — but only if the implementation is NOT also handed over,
  or TDD becomes theatre.

**Why:** `superpowers:writing-plans` requires the opposite, twice
("code blocks required for code steps", "Complete code in every
step"), on its stated assumption that the engineer has "questionable
taste" and "doesn't know good test design". That assumption does not
hold for a Sonnet implementer with an Opus reviewer behind it, and the
cost is concrete: the plan's copy of the code is the one that was
never run, type-checked or linted; it goes stale against the tasks
before it; an implementer handed the implementation cannot write a
genuinely failing test, which manufactures
[[tests-pass-for-the-wrong-reason]]; and the reviewer ends up
reviewing the planner's judgement laundered through the implementer
instead of independent work. Verification belongs in the gate — ruff,
mypy, pytest — not in an unverified markdown code block.

**How to apply:** when `writing-plans` asks for full code, write the
`Interfaces:` block verbatim and the steps as contracts, and say in
the plan that it deviates — the implementer needs telling there is no
code to copy, or they will look for it. Measured on B2: 874 lines of
code-carrying steps became 537 as contracts, about 1.6x, and its
worst task went from ~430 lines to ~190. Budget for that, not the 2.5x
I first guessed. A task whose contract exceeds ~400 lines is still the
split signal ([[spec-contract-size-predicts-pr-size]]).

Stating a step as a contract also catches errors writing it as code
hides: B2's task 1 test had asserted a fixed surviving line count,
wrong once already; as a contract it says *derive the count from what
came back and assert the ceiling*, which is the load-bearing claim.
Do not edit the skill in the plugin cache; upgrades overwrite it.

**Keep this as indexed memory, not a `CLAUDE.md` rule** — my human
partner's call, 2026-07-22: the full convention with its rationale is
far too long for a global `CLAUDE.md`, which every session pays for in
context whether or not a plan is being written. Recall covers it, since
it is only ever needed while planning. Do not re-propose promoting it.
