---
name: feedback_precommit_before_commit
description: "Always run `just precommit` before committing — never offer it as an optional suggestion"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 494dfa09-70b5-4ef8-bc70-603573015c26
---

Always run `just precommit` before any commit in this repo, and run it
unprompted — do not float it as "worth running before you commit" or ask
whether to. It is a required step, not a suggestion.

**Why:** offering a mandatory step as optional makes the user repeat
themselves; it reads as me dodging the work. The check is cheap and the
release recipe / gitlore precommitCommand depend on it passing anyway.

**How to apply:** after finishing edits and before (or as part of)
committing, just run `just precommit` and report the result. Same posture
as [[feedback_update_design_doc]] — treat the obligatory step as part of
the work, not a proposal.
