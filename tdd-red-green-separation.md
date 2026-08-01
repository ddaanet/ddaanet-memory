---
name: tdd-red-green-separation
description: "never bundle a red task into a green-task subagent in plan execution"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 321aa14b-160b-4491-b65f-58bb57249f27
  modified: 2026-07-30T16:01:05.823Z
---

When a plan splits a TDD pair into Task N (write failing test) and Task N+1
(implement, make pass), dispatch them as two separate subagents with the
spec/code-quality review gate between. Do not bundle them because they share
a commit at the end — the commit boundary is a packaging detail; the review
boundary is a correctness check.

**Why:** The red phase exists so an independent reviewer can confirm the
test actually fails against the unchanged code — i.e., that the test
exercises the behaviour the implementation will change. If the same
subagent writes the test and the implementation in one shot, that
confirmation is gone. The whole point of separating Task N from Task N+1
was to *force* the gate; collapsing them is rationalisation that "they're
tightly coupled" — and that is exactly the school-book pipeline failure
mode where TDD discipline gets traded away for throughput.

**How to apply:** When release-flow-style plans (or any plan from
`superpowers:writing-plans`) split a TDD
pair, dispatch:
1. **Task N implementer**: writes failing tests, verifies red, *does not
   commit*, reports DONE.
2. **Spec reviewer (red task)**: confirms tests cover the spec gap and
   actually fail.
3. **Task N+1 implementer**: applies the implementation, verifies green,
   commits combined diff per the plan's commit instruction.
4. **Spec reviewer + code-quality reviewer (green task)**: standard.

The relevant red flag thought is: "they share a commit, so let me
combine them." Treat that as a signal that the *test* is being treated
as ad-hoc scaffolding rather than as code under the same TDD discipline
as the implementation.
