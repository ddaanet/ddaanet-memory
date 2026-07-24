---
name: feedback_review_model_selection
description: Use Sonnet (not Haiku) for code review agents — weak models miss subtle issues
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 59a6338a-0de4-45c7-9150-a24136aca01c
---

Use Sonnet as the minimum model for spec and code quality reviewer subagents. Do not use Haiku for reviews.

**Why:** A reviewer weaker than the implementer can miss subtle issues — and that's true regardless of which model did the implementing. Haiku is too weak to be a reliable reviewer. User corrected this explicitly mid-session.

**How to apply:** In subagent-driven-development, dispatch reviewer subagents with `model: sonnet` (or higher). The implementer can use Haiku for mechanical isolated tasks, but reviewers must be Sonnet+.
