---
name: hook-output-channels
description: "no `injectToolCall`/`forceRead`/`requestTool`/`forceToolUse` field exists — only block, `updatedInput`, `updatedToolOutput`, `additionalContext`; `systemMessage` vs `additionalContext` vs stderr (`additionalContext` is honoured at PreToolUse too, spilled to a file past ~2KB, and injected text still leaves `Edit` needing a `Read`), `systemMessage` renders ANSI, what a hook ERROR does, exit 2 or JSON `deny` blocks the call not the turn (PreToolUse only), `ask` prompts the human in every mode — never the classifier, which passes out-of-repo `Edit`s; `updatedInput` alone reruns the full pipeline, `allow`+`updatedInput` bypasses the classifier over the rewritten command; `updatedToolOutput` must match the tool's output shape"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 618e80ab-c1a4-41c0-94c9-07e03fa4af68
  modified: 2026-08-27T11:47:42.000Z
---

What a Claude Code hook's stdout JSON can do, which channel reaches which
audience, and how a `PreToolUse` decision interacts with the permission
pipeline. Sections, by what a reader arrives holding:

- **Wanting a hook to force a `Read` or enqueue a tool call** → §1.
- **`systemMessage` / `additionalContext` / stderr / exit code** → §2.
- **`permissionDecision`, `updatedInput`, the auto-mode classifier** → §3.
- **`updatedToolOutput`, or correcting what an `Edit` reported** → §4.
- **Emitting file bodies the agent must then edit** → §5.
- **Choosing a channel, and why a blind agent re-verifies** → §6.
- **Wording a user-facing line, or an agent-facing deny** → §7.

## 1 · The control surface: no hook injects a tool call

No hook can inject a `tool_use`, force a `Read`, or enqueue a tool call. Only
the agentic loop generates tool calls. Hooks can block
(`PreToolUse.permissionDecision`, `Stop`/`PostToolBatch` `decision: "block"`),
rewrite input (`updatedInput`), rewrite a result (`updatedToolOutput`), and
inject text (`additionalContext`) — nothing more.

Verified 2026-07-22 against the shipped binary
(`~/.local/share/claude/versions/2.1.217`), not only the docs: zero occurrences
of `injectToolCall`, `requestTool`, `forceRead`, `injectTool`, `forceToolUse`,
`requestedTools`, `injectedToolCalls`. The four `triggerRead` hits are Node
stream internals (`stream._read = triggerRead`). Controls present in the same
grep: `additionalContext` 180, `updatedInput` 249, `permissionDecision` 34,
`PostToolBatch` 46, `InstructionsLoaded` 27, `systemMessage` 54.

**How to apply:** when you want file contents in the model's context
deterministically, don't try to make the agent Read them — have the hook read
them and emit the bytes as `additionalContext`. That is strictly better than a
forced Read: one round trip instead of two, no tool-permission surface, and the
selection stays in a script rather than in agent judgement. `additionalContext`
is honoured at `PreToolUse` as well as `PostToolUse` and arrives even when the
tool call fails (§2), so a deny-reason and the injected bodies can land in the
same turn. Two limits apply when the agent must *edit* what it is shown (§5).

`InstructionsLoaded` fires when `CLAUDE.md` and `.claude/rules/*.md` load
(matchers `session_start`, `nested_traversal`, `path_glob_match`, `include`,
`compact`) but is side-effects-only: no decision control, exit code ignored.

## 2 · Channel mechanics (verified 2026-06-10; PreToolUse additions 2026-07-17)

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
explicitly, report on `systemMessage`, exit 0. Applied in D17's index-sync
hooks.

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

**`systemMessage` renders ANSI escapes rather than showing them as literal
bytes.** A hook that speaks only when something went wrong can lead its line
with a style reset (`$'\033[0m'`) to escape the dimming applied to ordinary hook
chatter — handoff's drift report and context-size nudge both do.

## 3 · `PreToolUse` decisions and the permission pipeline

**`permissionDecision` (verified CC 2.1.246).** `deny` on stdout + `exit 0`
blocks the call like `exit 2` does, and additionally carries `systemMessage`;
either way it blocks *the one call, not the turn* — the agent gets the reason
and continues. `ask` forces a human prompt in every permission mode, auto
included: a hook cannot hand a decision to the auto-mode classifier, so an
`ask` that prompts too often is fixed by narrowing the hook's trigger, never by
expecting the classifier to absorb it. `allow` skips the prompt but not
`permissions.deny`/`ask` rules. `defer` exists but is print-mode-only and
solo-tool-only.

**`updatedInput` alone defers; `permissionDecision` settles (bundle read of CC
2.1.233, checked against 2.1.246–247).** `PreToolUse` hooks run before any
permission evaluation. `updatedInput` *with no decision* → the caller replaces
the working input and runs the **full** pipeline over the rewritten command:
deny and ask rules, the read-only auto-allow, then the auto-mode classifier.
`updatedInput` is schema-validated against the tool's `inputSchema`, and a
validation failure converts to a deny. `updatedInput` **with**
`permissionDecision: "allow"` → the caller treats the call as settled by the
hook; the only re-check is a narrow one for a matching deny rule, a matching ask
rule, or two fixed ask reasons, a `passthrough` from rule matching yields
`null`, and the classifier is never consulted. The two are separable and the
difference is the whole gate: **`allow` + `updatedInput` is a permission bypass,
not a rewrite.** A hook that inspects a *prefix* and rewrites — the shape of
every `cd`-restore or unsandbox helper — pre-approves an **arbitrary tail** it
never looked at. Emit `updatedInput` on its own and let the pipeline decide;
there is no turn cost, since the rewrite still saves the block-and-reissue turn
and a read-only tail is auto-allowed locally anyway.

Not probe-observable from inside a session: deny and ask rules are re-checked
under `allow` too, so neither discriminates, classifier verdicts are not
persisted, and a Bash `toolUseResult` carries no permission decision — the
bypass leaves no trace in the transcript. The durable check is a unit assertion
that the key is **absent**; an assertion written against the working output
will instead *require* the bypass. Two plugins shipped this:
`unsandbox-git-status` (retired 2026-08-26, confirmed live with
`true && git status --porcelain` running unchecked) and `cwd-safety` (FR5a/FR5c
restore rewrite, fixed 2026-08-28). See [[sandbox-effects]].

**The classifier scores safety, not convention** — it passes an unprompted
`Edit` of a pre-existing file in a sibling repo even against a CLAUDE.md rule
or a custom `autoMode.soft_deny` — so a convention-level gate on `Write`/`Edit`
needs a hook; only Bash-path writes are covered by the sandbox's write
allowlist. Evidence: prohibitions `docs/design.md`, "Off-project note drops
pass without asking".

**Obliging the agent to reach a checkpoint at all** is a separate mechanism
from injecting text: a `PreToolUse` deny on the first durable write of an
episode is self-scoping (no edit, no gate), where a `Stop` block would fire on
every turn including conversational ones.

## 4 · Rewriting a result: `updatedToolOutput`

**`updatedToolOutput` is validated against the tool's own output shape**, and a
mismatch is rejected with `PostToolUse hook returned updatedToolOutput that does
not match <tool>'s output shape` (string present in 2.1.232). So rewriting what
a tool reported is not a free-text channel: a hook that corrects, say, an `Edit`
result has to reproduce that tool's result object, not substitute prose for it.
Budget for establishing the shape before designing around this channel;
`additionalContext` carries a correction with no shape contract at all, at the
cost of leaving the original result in context beside it.

**For `Edit` that channel corrects nothing the model reads.** The result object
is `{filePath, oldString, newString, originalFile, structuredPatch,
userModified, replaceAll}`, but what reaches the model is a fixed string — `The
file <path> has been updated successfully. (file state is current in your
context — no need to Read it back)` — carrying no diff (uniform across 12,407
`Edit` results in the local transcript corpus; CC 2.1.232). So the agent's model
of an edited file comes from the edit it asked for, never from the tool's
report, and rewriting that report cannot move it. A hook that repairs the file
on disk restores agreement on its own; `additionalContext` is the only channel
that can say the bytes changed after the tool answered.

## 5 · Emitting file bodies the agent must edit

**Two limits make emitted bytes the wrong choice when the agent must *edit*
what it is shown.** A large `additionalContext` is not delivered whole: at
15.6KB the harness wrote it to a `tool-results/…-additionalContext.txt` file
and inlined only a ~2KB preview, so a hook emitting several file bodies
silently delivers a fraction of them unless the agent notices the pointer and
reads that file. And injected text does not register in the file-read ledger —
the ledger is keyed off actual `Read` calls — so `Edit` on a file whose bytes
arrived by `additionalContext` still fails until the agent Reads it, paying for
the content twice. There is no supported way to prime the ledger. Where the
recalled file is going to be edited, emit a directive naming the exact `Read`
calls instead of the bytes: two round trips, but no truncation and the file
lands editable.

## 6 · Which channel to reach for

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

**That redundant verification is the norm, not an anecdote, and it is worth
about two round trips per event.** Measured 2026-07-26 over ~2179 transcripts,
grouping `type:"assistant"` entries by `message.id` first: gitlore's
`memory-commit-batch.sh` commits the memory submodule on a file trigger and
reports success only on `systemMessage`. The agent then re-checked the outcome
after **62 of 68** successful commits (91%) — `ls` the IPC files, then
`git -C memory log`/`status` — costing a mean 1.97 extra assistant messages and
a median 6.4 s (quartiles 4.7 / 6.4 / 9.6) each. The control is the same repo's
other commit path, where the outcome comes back inside a Bash tool result the
model reads: **5 of 14** (35%). Rate tracks the channel, not the prompt — the
handoff directive that mentions checking was present for 58/64 of the blind
cases and absent for 4/4, with no difference. So when a hook acts on the agent's
behalf, pair the channels and **state the outcome as authoritative**; a blind
agent does not wait quietly, it goes looking. A hook's *instructional* branches
are worse than wasteful on the wrong channel — they are dead text: the same
script's deferred-retry branch fired 144 times and the agent never learned.

Per event: `UserPromptSubmit` → `additionalContext`. `PreToolUse` →
`additionalContext` or `permissionDecisionReason` (on deny); stderr + exit 2 also
surfaces. `PostToolUse` → `additionalContext`; stderr + exit 2 feeds back to the
model.

Consequence for gitlore (D14): route user-facing notices through `systemMessage`,
not stderr-on-exit-0 and **not** via the agent — `additionalContext` saying "tell
the user…" is model-dependent on the hot path, against NFR1/D7.

## 7 · Wording

`_wipe-emit.sh` is the in-repo reference implementation of the curt
user-facing style for `systemMessage` — read it rather than re-deriving the
conventions. The wording rules themselves — curt and informative for a
user-facing channel, no actionable phrases in an agent-facing deny, and the
scope limiter that exempts directive channels — are the
`craft:directive-writing` skill's, which states them at the moment they apply.
