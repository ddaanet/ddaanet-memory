---
name: feedback-reference-doc-scope
description: "a lasting guideline stays general/reusable (neutral examples, version- not repo-grounded); repo-specific application goes in a separate audit; confirm scope before delegating"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 4479f78d-488b-45fe-8ba1-d506de58b1bc
---

When asked to research best practices and produce a *lasting guideline*
(the point being to amortize generation + research cost across releases),
keep the guideline **general and project-agnostic**. Do NOT ground it in
the current repo or frame it as an audit input.

**Why:** ddaa intended the mypy/typing guideline to be reusable until the
toolchain changes. I scoped the Stage-1 research prompt to "ground in THIS
project" + "so an agent can audit scripts/*.py", which leaked ~19
"this repo does X" intrusions + a Repo-notes + Review-log section into a
doc meant to be durable. Repo-specific commentary makes a general
reference read as a stale one-off and won't transfer.

**How to apply:** Split the two artifacts cleanly —
- the **guideline** = general DO/AVOID rules, neutral illustrative
  examples (`Json = dict[str, Any]`, not `Entry` in `scripts/extract.py`),
  version-grounded but not repo-grounded;
- the **audit/application** = the repo-specific findings (the actual
  aliases, magic strings, config hygiene), point-in-time.
When a multi-stage research→review→audit pipeline is requested, the
research/review stages feed the *general* doc; only the audit stage is
repo-specific. Confirm intended scope (general vs. project-specific)
before delegating research. Related: [[feedback-design-doc-writing]],
[[feedback_subagents_over_applied]].
