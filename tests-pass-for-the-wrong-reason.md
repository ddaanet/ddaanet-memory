---
name: tests-pass-for-the-wrong-reason
description: "a green test asserting a true fact produced by the wrong cause"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 8bc34870-8b8c-49fa-85a0-4ad6d99085e8
  modified: 2026-07-30T15:34:20.850Z
---

Across the five tasks of ghmem dispatch B1 (2026-07-21), **every single
task shipped at least one test that passed for the wrong reason** — a
test that asserted something true, but true because of a cause other than
the code under test. Reading the tests never found them; mutating the
code they claimed to cover found all of them.

Concrete instances from that one plan:

- A `record_verdict` test called `serve_next` afterwards and asserted the
  next item was served — which "proved" the verdict was recorded. But the
  queue advances on the `Dispatch` row `serve_next` itself wrote, not on
  the verdict. Deleting the entire `record_routing_event` call left all
  ten tests green. The write was **completely unobserved.**
- A race test asserted the second reservation declined, the ledger held
  one entry, and the winner's row survived — all true of the conditional
  insert, but **equally true of the read-then-write the task existed to
  forbid.** A reviewer proved it by running the forbidden implementation
  against the test: green.
- A live-vs-cold-rebuild test compared `(Route, Dispatch, Topic)` tuples
  that were all empty in its scenario, so `live == rebuilt` was
  `() == ()` — vacuously true whether or not the reducer agreed with
  itself.
- Two note-validation tests raised the right `TriageSessionError` — but
  from the dispatch-ownership guard firing first, not the note check they
  were named for.

**Why:** a test that sets up state and asserts an outcome can be
satisfied by any cause that produces that outcome, and in a system with
several coupled writes (a ledger, a mirror, a queue that filters on a
different row) there is usually more than one. The assertion being true
is not evidence that the named mechanism produced it. Green is not
enough; green *for the stated reason* is the bar.

**How to apply:** do not trust a green suite on inherited or
subagent-written work — mutate the specific line the test claims to
cover and confirm a NAMED test dies (see
[[mutate-code-to-test-the-tests]]). When a test raises the right
exception, check nothing upstream raises the same type first. When it
compares collections, assert they are non-empty before comparing, or the
equality is vacuous. When it observes a downstream effect (the queue
advanced, the next item served), ask what else could produce that effect
and assert the actual write directly (`store.routes()`, the ledger's
last event) instead. In subagent-driven development this is not an
occasional risk — budget for one wrong-reason test per task and let the
adversarial reviewer tier hunt it ([[sdd-reviewer-tiering]]). It pairs
with the sibling lesson that briefs themselves misdescribe the code.
