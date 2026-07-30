---
name: stale-plugin-code
description: "running code ≠ what you edited: `/plugin update` no-ops if version unchanged; skill bodies need `/reload-plugins`; hooks.json freezes at session start (settings.local.json doesn't) — restart"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 618e80ab-c1a4-41c0-94c9-07e03fa4af68
  modified: 2026-07-24T19:21:20.941Z
---

One symptom, three causes: the plugin code actually running is not the code you
just edited. Nothing errors — the change simply appears not to work. **Before
trusting any verification of plugin code, confirm the loaded source matches the
repo.**

## Marketplace cache is keyed by version — `/plugin update` no-ops

`~/.claude/plugins/cache/<owner>/<plugin>/<version>/` is a flat file extraction
from a past push, not a git checkout (no HEAD). The dir is keyed by `<version>`,
so pushing new commits under the *same* version string leaves the cache stale and
`/plugin update` re-fetches nothing (confirmed live 2026-05-25: pushed under
`0.1.0`, update kept the old extraction; bumping to `0.1.1` forced the re-fetch).
Even after a refresh, only a *new* session picks it up — agent definitions in
particular are cached at session start.

**Fix:** ship via a **version bump**, not just a push. To develop against current
`main`, load with `--plugin-dir` instead of the cache.

Signature — discovered closing **Plan 04 Step 6** (2026-05-24): a marketplace
session was about to "verify" the **memory-merger** two-turn flow, but still had
the pre-flatten `commands/gitlore/` layout and the broken agent frontmatter (no
`name:`, no `allowed-tools:`) — it would have tested old code the repo had already
fixed (`e205aa6`, `27af3b8`). Check with `find
~/.claude/plugins/cache/<owner>/<plugin>/<ver>/commands` and by eyeballing
`agents/*.md` frontmatter. Tell in gitlore: a flat `/gitlore:resolve` in the skill
list means `--plugin-dir`/fresh cache; double-prefixed `/gitlore:gitlore:resolve`
means a stale nested cache.

Related: [[cc-agent-discovery]], [[cc-command-namespacing]],
[[dogfood-early]].

## Skill bodies are snapshotted at session start

Editing a plugin's own files mid-session does not make the change live in that
session. Observed 2026-07-20: after rewriting `skills/precompact/SKILL.md` and
committing it, invoking `/handoff:precompact` in the same session injected the
**pre-rewrite** body — even though the skill's reported base directory was the
working tree (`/Users/david/code/handoff/skills/precompact`) and the file on disk
held the new text. Content is snapshotted at
session start, not read at invocation.

**Fix for a skill body: `/reload-plugins`** — no restart needed. Run 2026-07-20 it
reported `18 plugins · 8 skills · 11 agents · 34 hooks`, and the next invocation
loaded the new body (the skill listing's `description:` updated too). The user can
invoke it with `! /reload-plugins`.

**Scope — this is verified for skill bodies only.** Whether `/reload-plugins`
re-registers a plugin's hook *events* is **inferred, not directly observed**: the
count it prints says it re-read the hook files, not that event routing re-bound. A
newly-declared hook event (a fresh `Stop` entry, a widened `SessionStart` matcher)
should be assumed inactive until a new session — and the one directly-observed
case went the other way (see the next section, where the fix was a new session).
Verify before relying on it either way.

Why it matters: developing a plugin *in* the session that uses it, a new magic
file can be written with no hook registered to consume it — a silent no-op, not an
error. Until a reload, fall back to the previously-shipped path and say so; never
report a mid-session plugin edit as "now active" without one.

Exception: bundled *scripts* invoked by a bare-name `bin/` shim take effect
immediately — PATH is set at session start but points at the live directory
([[skill-bundled-scripts]]).

## Hook event registration is frozen; script bodies are not

Claude Code re-reads hook configuration from `settings.local.json` **mid-session**:
a hook added during a live session fires on the very next matching tool call, no
`/clear` or restart. Verified 2026-06-10 (CC 2.1.x) — added a `PostToolUse
matcher:"Bash"` hook via an Edit, the next Bash call fired it with that call's
`tool_input`. This is how D15's `EnterWorktree|ExitWorktree` matcher was confirmed
without a restart ([[cc-worktree-memory-freeze]]). Timing caveat: a
`PostToolUse` hook fires *after* the triggering tool returns, so a single Bash call
cannot observe its own hook — check the side effect on the next call. (Distinct
from `autoMemoryDirectory`, which genuinely resolves once at startup and no hook
can change — [[auto-memory-directory]].)

A **plugin's own `hooks/hooks.json` does not reload**, even under `--plugin-dir .`
where the plugin loads straight from the checkout you are editing. Observed
2026-07-15 (CC 2.1.210) moving gitlore's `index-sync-post.sh` from
`PostToolUse(Write|Edit)` to `PostToolBatch`: after editing `hooks.json`, a live
`Edit` to `memory/MEMORY.md` fired the unchanged `PreToolUse` hook and left its
stash, but the post hook never ran as `PostToolBatch` — no frontmatter rewrite,
stash unconsumed.

The split that makes this confusing:

- **Script bodies are read per invocation** — edit a hook script and the next
  firing runs the new code. The rewritten script was live.
- **Event registration is read at session start** — so the *code* was new while
  the *event routing to it* was old. The hook got a `PostToolUse`-shaped payload,
  found no `.tool_calls[]`, and silently no-op'd.

Signature: a hook that looks dead for no reason right after you move it to a
different event. It isn't dead; it's being called on the old event with the old
payload shape. **Don't debug the script — start a new session.**

Corollary for probing: you can test a hook-*logic* change in the session that makes
it, never a hook-*event* change. Routing the test through `settings.local.json`
instead is blocked by the auto-mode classifier as self-modification, so that needs
the user's explicit go-ahead.

Not verified: whether the *old* registration still fires after the edit. A no-op
and a non-call look identical from outside.

See [[hook-output-channels]].
