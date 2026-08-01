---
name: harness-over-agent
description: "first rule of skill design: where deterministic mechanical work belongs"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 559f871e-4271-4a30-a460-4ed106886f7e
  modified: 2026-07-24T17:52:14.545Z
---

Detection logic, branching, and decision-making should live in a script or hook. The agent reads structured output and acts on it — it does not reason its way to a decision when a mechanical check would do. First rule of skill design: don't make the agent do what the harness can do.

**Why:** Scripts/hooks are deterministic and testable; agent compliance with skill-body instructions is not, and doesn't vary with model version or context pressure. Applies to hook-manager detection, remote-provider detection, merge-state checks, worktree detection, and any "always run this command" step. In one incident the user pushed back when an in-skill `rm -f` cleanup step was proposed, then stated the rule directly: splitting mechanical logic between the skill body and the harness also obscures the mechanism — keeping it in the harness puts each piece in one place.

**How to apply:** When implementing any detection or decision flow, write a script/hook that outputs a structured result; the skill or agent acts on that output rather than inspecting files and deciding itself. When writing or editing a skill, reach for hooks + scripts first for anything expressible as a deterministic file operation, path check, or trigger — reserve the skill body for parts that require judgement (deciding what to write, what the residual is).

Related: [[test-automation-8020]] (same principle, applied to test coverage).
