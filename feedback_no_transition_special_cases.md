---
name: feedback_no_transition_special_cases
description: "Don't code around a version transition in a tool with a user base of one — fix forward, one fact plus one act"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: e114f08e-9b78-4cde-9cc6-f14fd80f67ed
  modified: 2026-07-29T16:18:51.066Z
---

Don't add a branch, fallback, or diagnostic narrative that exists only to
cover a transition between versions of tooling you also control — a plugin
update that has not re-pinned its config key, an old on-disk layout, a
half-migrated path. Fix forward instead.

**Why:** David: "Don't write a special case for the gitlore transition, it
has a user base of one." With one user the transition window is a single
restart; the branch outlives the case it covers, and its explanatory prose
is wrong the moment the transition ends. Same instinct as
[[feedback_no_compat_aliases]] — carrying the old state costs more than
patching through.

**How to apply:** When a message or code path starts enumerating causes
("installed before this session / updated since / the pinned path moved"),
collapse it to one fact plus one recovery act. Reserve real migration
handling for boundaries with users you do not control. Read-only *liveness*
detection of a foreign tool's state is a different thing and still applies
([[feedback_detect_liveness_not_presence]]).
