---
name: feedback-skill-description-purpose-first
description: "skill `description:` frontmatter is TUI-visible; lead with what it does, then triggers/\"Use when\", then exclusions; never a bare trigger list"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 246af36f-a1b2-4592-8c6f-7170974482af
---

A skill's `description:` frontmatter is shown verbatim in the Claude Code
TUI skill list, so it must **open with a concise statement of what the
skill does/is**, then follow with the trigger phrases / "Use when…"
clause, then optional exclusions. Never lead with a bare trigger list —
that reads as noise in the TUI and the user can't tell what the skill is
for at a glance.

**Why:** the description is the at-a-glance summary the user reads when
choosing a skill; a trigger-first description fails that job. Confirmed
against marketplace skills (revdiff: "Review diffs… Activates on…";
skill-creator: "Create new skills… Use when…"; mcp-builder: "Guide for
creating… Use when…").

**How to apply:** structure every skill description as `[one clause:
what it does] + [triggers / when-to-use] + [optional: what NOT to use it
for]`. Also drop overspecific or stale implementation detail from the
tail (e.g. "writes a short markdown task file… otherwise leaves nothing
in place") — mechanics aren't a useful TUI summary. Applies to all
skills in a plugin, not just the headline one. See [[project_claude_plugin_dev]].
