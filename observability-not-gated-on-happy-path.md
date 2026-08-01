---
name: observability-not-gated-on-happy-path
description: "failure-reporting for the paths the run you watched didn't take"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 94e0a066-a634-4a25-a32d-b0f53b992c25
  modified: 2026-07-20T20:04:25.835Z
---

Do not defer or skip a failure-reporting mechanism on the grounds that the
run you just watched went fine. "Add the breadcrumb only if this run's submit
looks marginal" is a category error: a clean run is evidence about behaviour,
never about observability. The run that needs the breadcrumb is the failing
one, and it is precisely the run nobody is watching.

**Why:** caught in the handoff plugin (2026-07-20). I had written that open
decision into a task file and was about to ship on it; the user's question —
"does that mean that if the watcher fails somehow, it's completely invisible
to the agent?" — was what forced the audit. Enumerating the failure paths found
one that was silent to *both* the agent and the user, which no amount of
watching successful runs would ever have surfaced.

**How to apply:** when weighing an observability fix, enumerate the failure
paths and ask who learns about each one, instead of reasoning from how the
happy path looked. Rank them: silent-to-everyone is the one that must be
fixed; visible-to-the-user-but-not-the-agent is weaker; and a path where the
signal would be inferred rather than observed is worth leaving silent if
inferring it produces false alarms. Related: [[verify-at-source]]
(reproduce before churning) is the mirror case — that one says don't act on a
weak signal, this one says don't decline to act because the signal is absent
by construction.
