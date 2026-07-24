---
name: reference-hook-output-channels
description: "systemMessage = user-visible; additionalContext = model-only, honoured at PreToolUse too (arrives even when the call fails); stderr only on non-zero exit or `--verbose`; hook ERROR = systemMessage + exit 0; only exit 2 blocks, PreToolUse only"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 618e80ab-c1a4-41c0-94c9-07e03fa4af68
  modified: 2026-07-24T19:21:51.382Z
---

## Channel mechanics (verified 2026-06-10; PreToolUse additions 2026-07-17)

- **`systemMessage`** (top-level field) — the user-visible channel. Works for the
  events gitlore uses (`SessionStart`, `PostToolUse`); gitlore's SessionStart
  launcher-guard warning rides it. This is the field for anything the human
  should see.
- **`hookSpecificOutput.additionalContext`** — injected into the model's context
  only; **never echoed to the user**. Silent by design. Honoured on
  **`PreToolUse` as well as `PostToolUse`** (both verified 2026-07-17 via nested
  `claude --print` + `--settings` scratch hooks): it arrives as a
  `<system-reminder>` attributed to `<event>:<Tool> hook`, and on `PreToolUse` it
  is delivered **even when the tool call then fails** — so a hook can annotate a
  call it has rewritten without depending on the command succeeding.
- **stdout** — consumed as JSON; not echoed. **Parsed ONLY on `exit 0`.**
- **stderr** — ignored on `exit 0`; on non-zero exit, shown to the user only with
  exit code **2**, or with `--verbose` for other codes (a "hook error" label
  appears). `SessionStart` is non-blocking — continues regardless of exit code.

**Reporting an ERROR needs `exit 0`, not merely tolerates it (2026-07-15).** The
naive "fail loudly ⇒ exit non-zero" is backwards: stdout JSON parses only on
`exit 0`, so a non-zero exit **discards the `systemMessage`** — the only reliably
user-visible channel — leaving stderr (exit 2 / `--verbose` / debug log). The
honest error path is **`systemMessage` + `exit 0`**, which is what D14 concluded
when it moved SessionStart's errors off `stderr` + `exit 1`. `|| true` and a bare
`|| exit 0` on a fallible command are dishonest error paths — check status
explicitly, report on `systemMessage`, exit 0. Applied in D17's index-sync hooks;
see [[feedback_prohibition_vs_procedure]].

**Blocking asymmetry** (don't conflate "don't exit 2" with "don't error"):

| event | can it block? |
|---|---|
| `PreToolUse` | **`exit 2` blocks the tool outright.** Other non-zero → action proceeds + "hook error" notice. |
| `PostToolUse` | **Nothing can block** — docs: *"PostToolUse hooks can't undo actions since the tool has already executed."* `exit 2` is mere feedback to the model; the exit code buys nothing. |

So "exit 0 on every path so we never block a Write" is over-broad reasoning that
lands on the right answer: `exit 0` is right for *visibility*, not for
non-blocking.

**Unresolved tension — re-verify before relying on it.** The 2026-06-10 finding
says stderr reaches the user only on exit 2 / `--verbose`. The docs (fetched
2026-07-15, `code.claude.com/docs/en/hooks-guide.md`) instead describe *any*
non-zero as showing a `<hook name> hook error` notice + the first stderr line in
the transcript, with full stderr to the debug log. Possibly a CC behavior change,
or "transcript" meaning the transcript view rather than the inline UI. The
conclusion holds under both readings — `systemMessage` + `exit 0` — but the
stderr claim itself is stale-suspect.

## Which channel to reach for

Decide explicitly who the audience is. User-only → `systemMessage`. Agent-only →
`additionalContext`. Both → emit both fields in the same hook JSON. Don't assume
`systemMessage` is general-purpose.

A hook that emits only `{"systemMessage": "..."}` is **invisible to the agent**.
If the agent needs to know what the hook did, it must get an agent-facing
channel, or it redundantly verifies with `ls`/`cat`. Observed live: handoff's
`prompt-pre-hook.sh` and `skill-pre-hook.sh` emitted only `systemMessage` when
wiping prior handoff files; the wipe ran correctly but the agent then ran
`ls .claude/handoff-task.md .claude/handoff.md` to check — exactly the wasted
work the hook existed to avoid.

Per event: `UserPromptSubmit` → `additionalContext`. `PreToolUse` →
`additionalContext` or `permissionDecisionReason` (on deny); stderr + exit 2 also
surfaces. `PostToolUse` → `additionalContext`; stderr + exit 2 feeds back to the
model.

Consequence for gitlore (D14): route user-facing notices through `systemMessage`,
not stderr-on-exit-0 and **not** via the agent — `additionalContext` saying "tell
the user…" is model-dependent on the hot path, against NFR1/D7. See
[[feedback_harness_over_agent]], [[feedback-verify-at-source]].

## Wording the user-facing channel: curt and informative

`systemMessage` and similar user-facing output: **lowercase, em-dash separator,
factual, no instructions or actionable phrases.** One short line — enough for the
user to know what happened, no more. The user named this style in the 2026-05-19
SessionStart-hook brainstorm ("style for user visible systemWarning (and similar)
is curt and informative"). No "to fix this, run X" — that is the agent's job via
`additionalContext`, not the user's. Matching examples:

- `handoff loaded — 3.2 KiB, saved 8m ago`
- `handoff: wiped prior handoff-task.md, handoff.md`
- `handoff extracted to /path/to/handoff.md`

`_wipe-emit.sh` is the in-repo reference implementation of this style — read it
rather than re-deriving the conventions.

## Wording an agent-facing DENY: no actionable phrases

When `permissionDecisionReason` (or any agent-facing message) denies a tool call,
never include phrases like "stop and report it to the user" (agent tries to
invoke a reporting tool), "if you really need X, run Y" (agent treats it as
authorisation), "revert via git rather than editing here" (agent runs git
revert), or the canonical "remove the lock if stale" (agent removes the lock
immediately).

**Why:** agents lack the contextual filter that lets a human read diagnostic
prose as descriptive rather than prescriptive. Any imperative phrasing in a
deny-reason gets executed literally. Distinct from harness-over-agent: even when
the work is correctly in a hook, the *wording* of agent-facing output still needs
care.

**How to apply:** phrase deny reasons as unconditional refusal + redirect to the
legitimate path ("the release recipe owns this — invoke it instead of editing the
file"). Never offer escape hatches the agent can self-authorise. Put human-facing
detail in a separate curt `systemMessage`. If you write "if you … then …" in an
agent channel, rewrite as "do not …".

**Scope — this rule governs DENY channels, not DIRECTIVE channels.** A
`PreToolUse` deny is meant to *stop* the agent, so any actionable phrase is
harmful. But `additionalContext` used as a *directive* — where the whole point is
that the agent acts (present a line to the user, summarize+commit per a probe's
output) — is the opposite case: **imperative wording is correct and required
there.** Do not over-apply. Tell: is this channel trying to halt the agent (deny
→ descriptive only) or drive it (directive → imperative is fine)? A reviewer
flagged handoff's non-tmux `/rename` `additionalContext` as a violation; it was
not — that channel directs the agent to relay the line, and the probe's own
summarize→approve→commit directive is equally imperative by design.
