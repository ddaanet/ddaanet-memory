---
name: dont-bake-in-guarantees
description: "a guarantee a later caller must relax belongs in an opt-in method, not every call (fsync → `Ledger.sync()`)"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 0f51d69f-3ca2-4d5f-915c-74d76b788571
  modified: 2026-07-30T15:34:24.386Z
---

Don't design an API that is "buggy by design" by baking in a strong
guarantee a foreseeable caller will need to relax later. When a stronger
promise (durability, ordering, atomicity, validation) conflicts with a
known future need (throughput, batching), don't hardcode the strong
version into every operation — expose the guarantee as a separate opt-in
method the caller invokes at its own checkpoints.

**Why:** removing a guarantee from a published contract is a breaking
change and a design smell; adding an opt-in later is not. A weaker
default + explicit escalation is honest about what each call promises.

**How to apply:** concrete case — ghmem's `Ledger.append` used to fsync
per event, which would throttle Part 3's bulk fetch. Fix: `append` only
flushes to the OS (crash-visible), and a new `Ledger.sync()` delegates
to `os.fsync` when the caller wants power-loss durability.
