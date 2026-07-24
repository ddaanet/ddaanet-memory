---
name: feedback_test_validation_by_mutation
description: How the user requires after-the-fact tests to be validated when not written via TDD
metadata: 
  node_type: memory
  type: feedback
  originSessionId: dfaee0d7-8d15-4a23-b355-d8691f314eb2
  modified: 2026-07-24T14:55:02.238Z
---

When tests are written *after* the implementation (not via TDD), validate each one by mutating the code so the test fails (turns red), then reverting — proving the assertion actually bites.

**Why:** a test written against already-green code can be vacuous; seeing it go red under a deliberate fault is the only proof it tests something real.

**How to apply:** once the suite is green, for each test introduce a targeted fault in that test's target code, confirm that specific test (and only the suite) goes red, restore, and end green. Report the red→green evidence (e.g. a per-test table). Relates to [[prefer-bats-for-bash-tests]].
