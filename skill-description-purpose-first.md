---
name: skill-description-purpose-first
description: "a skill or command `description:` is TUI-visible *and* injected every session: purpose, then \"Use when\", then exclusions; a plugin command costs context exactly like a skill (both under `/context`'s Skills), so length is the lever, not command-vs-skill"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 246af36f-a1b2-4592-8c6f-7170974482af
  modified: 2026-08-17T18:55:53.602Z
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
skills in a plugin, not just the headline one. See [[claude-plugin-dev]].

**A description is injected, not merely displayed, and a plugin command pays
the same.** Every enabled skill's and command's `description:` loads into
context at session start, so its length is a per-session cost in every repo
where the plugin is enabled. `/context` reports the whole set under *Skills*,
and a plugin command with no `skills/` directory of its own appears there
beside them — `gitlore:install`, one short clause, under 20 tokens;
`gitlore:add-tier`, ~70.

That matters past style. Moving an always-loaded line — a memory index entry,
say — into a command rather than a skill does not make it free: the lever is
description length, not the command-versus-skill choice, and a description
carrying the routing surface the index line carried is a lateral move.

Model guidance does not belong in a skill body either. Which model runs the
work is an agent-frontmatter concern, so a skill that says "run this on Sonnet"
is stating something it cannot enforce — put it on the agent definition
instead.
