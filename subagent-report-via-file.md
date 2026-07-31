---
name: subagent-report-via-file
description: Subagent final messages can vanish in transit; have reviewers write their report to a file and reply with only a pointer
metadata: 
  node_type: memory
  type: feedback
  originSessionId: de3d2151-a51d-4e13-8bd4-cee0e4c48775
  modified: 2026-07-29T16:04:21.302Z
---

A subagent's final message is not a reliable delivery channel: an agent can
finish its work and return nothing but an idle notification. The artifact is
the deliverable; the reply is only a notification that one exists.

**Why:** an empty reply is indistinguishable from a failed agent, so the
reflex is to re-dispatch — paying twice for the same analysis and, for
implementers, risking duplicate commits. The transcript is not a fallback: for
local agents it is the full JSONL conversation, and reading it overflows the
context window. A named output file survives both problems, and keeps a long
report out of the dispatcher's context besides.

**How to apply:** every dispatch whose output matters names an output file,
instructs the agent to write the report there *before* composing its reply,
and asks for a reply of one line pointing at it. Read the file, not the
message. An agent that goes idle with nothing has usually done the work —
check the artifact, `git log`, and the working tree before concluding
otherwise; a nudge naming what is already visible is cheaper than a
re-dispatch. See [[tests-must-go-red]] for the same habit applied to
claims: trust the observed artifact over the report of it.
