---
name: sdd-durable-state-across-compaction
description: keeping subagent-driven-development state durable — recover dead subagents from staged work; flush the SDD ledger (not just memory) before compaction
metadata: 
  node_type: memory
  type: feedback
  originSessionId: d015e0f1-8fa3-4170-9152-2c9265c45dfc
---

Two failure modes threaten subagent-driven-development (SDD) state, both
seen on the ghmem Part 3 run (2026-07-16). Guard against both.

**1. An implementer subagent can die after staging but before committing
or reporting.** The ghmem T2 implementer finished — wrote its report,
`git add`-ed everything, ran `just green` — then the process vanished
without a commit or a final status message. Recovery: confirm the agent
is gone (SendMessage → "no agent reachable"), verify the green state
YOURSELF (re-run the suite — don't trust a dead agent's staged snapshot),
then commit the staged tree unchanged. Do NOT re-dispatch the task — the
work is done and on disk. Mitigation for future dispatches: require the
subagent to `SendMessage` its final report to "main", not just go idle —
an idle notification carries no report.

**2. `precompact` flushes memory only — it does NOT touch the SDD
ledger.** The skill's own words: "precompact is the memory flush alone;
the task crosses compaction in the summary." But SDD explicitly distrusts
the summary ("controllers that lost their place re-dispatched entire
completed task sequences… trust the ledger over recollection"). So
mid-SDD-run, before any compaction, expand the ledger yourself: record
each task's review verdict, the open findings + their resolutions, any
in-flight fix-wave, and how to VERIFY an incoming subagent report — so the
post-compaction session can act on it. This is SDD "Durable Progress"
discipline, not a precompact step; precompact won't prompt it.

**Why:** losing SDD place is the single most expensive failure in the
workflow. The compaction summary paraphrases; git and the ledger do not.

**How to apply:** keep the ledger current at every task seam and
especially before compaction; recover dead subagents from git + staged
state, never by re-running completed work. See [[sdd-reviewer-tiering]],
[[no-volatile-git-state-in-memory]].
