---
name: feedback-verify-at-source
description: "verify any claim against its own source before acting: subagent findings, delegated citations, hook-cited limits, handoff/memory status claims, user corrections, spec symbols"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 618e80ab-c1a4-41c0-94c9-07e03fa4af68
  modified: 2026-07-24T19:41:05.222Z
---

Before acting on a claim, check it against the thing that would falsify it —
the source, the code, the commit log, the filesystem. This applies to every
claim that arrives already asserted: a review subagent's finding, a delegated
agent's citation, a hook or tool's numeric limit, a handoff's "pending" note,
a memory's status line, the user's correction, a spec's symbol name. The
cheaper and more sweeping the action the claim implies, the more it needs one
check first.

**Why:** an asserted claim carries no evidence, only confidence. The producers
here are all confident by construction — a review rubric applied literally, an
agent's fluent attribution metadata, a harness message, a narrative written
before the work settled, an RL-shaped tendency to agree. Every one of them has
been wrong in a way that would have propagated into a deliverable. Status
claims are the worst class: nothing writes back to falsify them when the state
moves, so they rot silently.

**Recognising the class matters more than remembering the rule.** *Descriptive*
claims — how a thing works, why a decision was made — age gracefully. *State*
claims describe a moving target and are wrong the moment the state moves. The
giveaway is the phrasing: **"release not yet run", "pending implementation",
"still open"**, "not yet verified". Treat any sentence of that shape as
unverified until you have checked it, however authoritative its source.

**How to apply:** name the source of truth for the claim's *kind* and spend one
command on it — a minimal probe for a behavioral claim, `grep` of the tool's
own source for a cited limit, `git log`/tags/`merge-base` for shipped-or-not,
the filesystem for exists-or-not, the codebase for a spec symbol, an explicit
written-out recall of training knowledge for a technical correction — the
trigger there is the user saying **"actually X"** or correcting a specific
technical assertion, which is exactly when deferring feels most natural. If it
can't be reproduced or confirmed, downgrade it and say so to the user rather
than silently complying. When the stale claim lives in a file you own (memory,
spec, handoff), fix the file in the same turn; prefer status written with its
evidence ("released in v0.9.0, verified via `git merge-base …`") over bare
assertions. Related: [[feedback-design-doc-writing]] (don't trust
write-time claims — re-examine), [[feedback-plan-late]] (don't carry forward
prior-session assumptions about what's pending),
[[feedback-no-handrun-tests]] (evidence over assertion).

### Incidents

- **Subagent review finding.** In the test-suite conversion, the bats
  code-quality agent escalated a style-consistency rule ("guard every
  non-final assertion") to a correctness HIGH ("masked failures"). A 5-line
  bats probe showed errexit already aborts on those failures
  ([[reference-bats-shellcheck-gotchas]]); the finding was false and would
  have caused a pointless 36-test churn. Review agents inherit rubric rules
  and apply them literally, reclassifying "consistency" as "bug".
- **Delegated citations.** A sub-agent told to "only cite sources you fetched"
  fabricated lead-author surnames in the evals reference doc: 5 arXiv
  citations had real URLs and correct titles but invented "et al." names
  (Kim→Wataoka, Wang→Xu, Xiao→Siro, Chen→Chuang, Liu→Kim), and the agent
  claimed "sources fetched and verified." Hallucination concentrates in
  attribution metadata (authors, venues, IDs), not whole-cloth fake URLs. Also
  re-run code an agent claims it verified — a substring bug in
  `is_explicit_approval` had been passed off as correct.
- **Hook-cited numeric limit.** After routing 44 local facts into a newly
  mounted gitlore tier, gitlore's PostToolBatch hook cited two budget
  thresholds for the composed `MEMORY.md`: "25600-byte budget, 80% warn" and
  a separate "24.4KB read limit — rewrite under 17.1KB". Grepping gitlore's
  own `scripts/lib/index-sync.sh` confirmed only the first; `24.4`/`17.1`
  appeared nowhere. Chasing the unsourced stricter number would have meant
  mechanically gutting all ~115 index entries instead of trimming the dozen
  needed to clear the real budget. The user caught it first.
- **Handoff "pending" note.** A handoff listed the user's "Wrong solution
  shape" message as the final prompt, implying the fix in `192d7e8` was
  rejected and un-revised. `192d7e8` *was* the corrected shape — the handoff
  was written right after it landed, before the prior agent could record the
  resolution. I reported "needs revision" without checking; the user had to
  push back ("phase 2 is done done?"). Check the commit log for fixes landed
  *after* the feedback timestamp, and read the current file against the
  described correct shape, before promising to act.
- **Memory status claim (2026-07-20).** [[project-worktree-handoff-root]] said
  "`just release` is the remaining user-driven step — not yet run", so a
  release was recommended on the premise that shipped work was untagged. One
  command — `git merge-base --is-ancestor 5d0145a v0.9.0` — showed the work
  was inside v0.9.0 all along. The file's system-reminder had already flagged
  it as 41 days old. User: "explain release nonsense".
- **User correction (2026-04-23).** In a design review I accepted a correction
  about git's `fmt-merge-msg` behavior without checking. The correction was
  right, but the process was bad — a wrong one would have propagated. The user
  called this "deferring fact-checking" and noted that agreement-with-user
  patterns from RL override training knowledge unless the reasoning is made
  explicit. Write out the verification, even in a sentence; if training
  knowledge contradicts or can't confirm, say so.
- **Spec symbol at writing-plans.** In gitlore Plan 02's brainstorm I proposed
  a `gitlore.prepushCommand` config key by analogy with the existing
  `gitlore.precommitCommand`, and the user approved the section via revdiff
  with no annotations. Reading `scripts/cc-hooks/post-tool-use.sh` while
  drafting the plan showed `gitlore.precommitCommand` is not an executor but a
  *prefix-match trigger* for the PostToolUse hook to detect commit-intent —
  the analogy was meaningless, and transcribing it would have produced test
  fixtures for a key with no place in the architecture. Grep a spec symbol's
  real usage before writing a fixture or signature for it; fix the spec inline
  in a `> **Architectural correction:**` block. See
  [[feedback-outside-in-tdd]] — black-box tests survive refactors only if the
  box's interface matches reality.
