---
name: test-automation-8020
description: "If you've spent 80% writing tests, the marginal 20% to automate them is small — don't propose agent-driven tests when assertion-based shell tests work"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 45acd379-a2c1-4cfc-8f7b-e506ad744d38
---

If you've already spent the effort to write test cases (80%), the
**marginal 20% to automate them is small** and pays back forever.
Don't propose agent-driven tests ("run it and inspect the output")
when assertion-based shell tests work — and they work for most
plugin-level behaviours.

**Why:** in the 2026-05-19 handoff design review, the user annotated
the spec's Testing section: "are those automated tests, or
agent-driven? if you take the time to write tests, take the remaining
20% (that is a budget 1/4 the test-writing effort) to automate tests.
how does that sound?" The reframing is useful: automation is a 25%
markup on test-writing time, not a separate undertaking.

**How to apply:**
- When writing a Testing section in a spec or plan, state explicitly
  that tests are automated (assertions, exit codes, CI-propagated).
- Use the existing `tests/*.sh` harness (assertion functions,
  pass/fail summary, exit-code propagation) — the marginal cost of
  adding a new scenario to an already-automated suite is tiny.
- Reserve "manual/eyeball" testing for things that genuinely resist
  assertion: TUI rendering, screenshot diffs, smoke output on real
  data ([[jsonl-sidechain-segregation]]-style smoke tests are
  okay — they're documenting reality, not asserting it).

Related: [[harness-over-agent]] (same principle, broader scope —
deterministic work belongs in the harness, not the agent).
