---
name: First rule of skill design — harness over agent
description: For mechanical, deterministic work in this plugin, prefer hooks over instructions in the skill body — agent compliance is a weaker guarantee than a hook
type: feedback
originSessionId: de10cc41-7b54-42ff-a641-b371a3b7ed4e
---
First rule of skill design: don't make the agent do what the harness
can do.

**Why:** The user pushed back when I proposed an in-skill `rm -f` step
for cleanup, then explicitly stated this rule. Hooks are
deterministic; agent compliance with skill text is not. Splitting
mechanical logic between the skill body and the harness also obscures
the mechanism — keeping it in the harness puts each piece in one
place.

**How to apply:** When writing or editing a skill in this plugin
(`skills/handoff/SKILL.md`), reach for `hooks/hooks.json` +
`scripts/*.sh` first for anything that can be expressed as a
deterministic file operation, path check, or trigger. Skill body is
for the parts that require judgement (deciding what to write, what
the residual is). If a step boils down to "always run this command",
that's a hook, not a skill instruction.
