---
name: brief-confirm-option
description: "When a brief ranks options and recommends one, confirm the choice with David before implementing it"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 7148d8ea-7e2c-458c-a3e6-65537aca52f6
  modified: 2026-07-29T16:07:45.924Z
---

A brief that enumerates options and recommends one has not decided anything.
Surface the choice and get David's call before writing code, even when the
brief's own stated constraints appear to settle it.

**Why:** briefs here are usually written by an agent in another repo, so both
the recommendation and the constraints justifying it are that agent's
assumptions rather than verified requirements. A constraint can be flatly
untrue — "backward compatibility matters more than elegance" reads as decisive
and is the opposite of how this toolkit is designed
([[plugin-dev-no-backcompat]]). Implementing the recommended option on that
basis costs a full cycle and lands the wrong design.

**How to apply:** treat a brief's preference order and constraint list as
input. Verify the recommendation cheaply — empirical probes are worth running
whichever option wins — then ask which to implement. Keep doing the
non-contingent work while waiting rather than blocking outright.
