---
name: no-internal-back-compat
description: "don't keep backward compatibility for internal clients; change the callers instead"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: ab541f5f-d68e-4918-917d-53fd4779ebab
---

Don't preserve backward compatibility for internal clients. When
something changes — a symbol moves, a signature shifts, a name changes —
update the in-repo callers to match rather than leaving a compatibility
layer that spares them the change.

**Why:** internal clients are not a public API surface. Anything kept
only to avoid touching callers is dead weight that hides the true
dependency and lets code keep referencing the old shape indefinitely. My
human partner called this out when a `RoutingEvent` re-export lingered in
`store.py` purely so existing ghmem tests kept resolving it.

**How to apply:** find every caller in the repo and change it; remove
whatever only existed for compatibility. Reserve compatibility work for
genuine external/published API boundaries.

See [[no-code-for-impossible-cases]], [[dont-bake-in-guarantees]].
