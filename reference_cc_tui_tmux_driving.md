---
name: reference-cc-tui-tmux-driving
description: "mid-turn input has FOUR classes (TUI-local intercepted / harness-action queued / plugin-command own-turn / **prose injected into the running turn**); Stop + SessionStart(compact) are the authoritative gates; Stop doesn't fire on Esc (verified v2.1.215)"
metadata:
  node_type: memory
  type: reference
  originSessionId: 8070a865-ea66-486b-9d50-a946add0a54c
  modified: 2026-07-20T19:05:43.571Z
---

Verified 2026-07-19 against Claude Code v2.1.215 (Opus 4.8, tmux 3.5a) by
driving a throwaway `claude` session in a detached tmux session with a logging
hook on every event. Supersedes the 2026-07-18 v2.1.214 pass, whose
"input during busy is queued and runs as its own turn afterward" was too
coarse — it holds only for slash commands, not prose.

**Mid-turn input has four distinct classes** (not one "queued" behavior):

| typed mid-turn | behavior | UserPromptSubmit |
|---|---|---|
| `/focus` (TUI-local) | intercepted immediately, never queued | — |
| `/compact` (harness action) | queued → interpreted at turn boundary | no |
| `/handoff` (plugin/skill) | queued → drains as its own turn | **yes** |
| plain prose | **injected into the running turn's next call** | no |

- **Slash input never reaches the model as prose.** Unrecognized commands drain
  to `● Unknown command: …`. No hallucination path.
- **Prose is the dangerous input** — it contaminates the running turn.
- **Queued input does NOT bypass hooks in general.** `/compact` skips
  UserPromptSubmit because it never becomes a prompt; a queued plugin command
  fires it with the raw command text as `prompt`.
- **Hook chain:** `Stop` → `PreCompact` (+0.75s) → `SessionStart` with
  `"source":"compact"` (+47s). Queue drains ~0.75s after `Stop`.
- **`Stop` does NOT fire on Esc interrupt** — interrupted turns fail safe.
  Payload carries `last_assistant_message`, `stop_hook_active`, `prompt_id`,
  `effort`, `background_tasks`, `session_crons`.
- **Command recognition is visible before Enter**: typing `/compact` with no
  Enter renders an autocomplete row + description; `/comzzz` renders
  `No commands match "/comzzz"`. Makes type-verify-submit implementable.
- **Busy = the `[0-9]+s ·` timer** + randomized gerund. `_rename-lib.sh`'s
  `is_busy` transfers unchanged, BUT read only the **visible** pane —
  `capture-pane -S` history matches a stale timer and reports busy long after
  `Stop`.
- **An Enter sent immediately after a long `send-keys -l` is absorbed as a line
  break, not a submit** — it lands inside the TUI's paste window. Observed
  2026-07-20: `/compact` (line 1) submitted because its command-recognition
  readback happened to sit between the text and the Enter; the continuation
  prose (line 2, ~190 chars, no readback) did not, and needed a manual Enter.
  **Always put a settle delay between the literal send and the first Enter.**
- **Verify a submit with `is_busy`, never "the composer looks empty".**
  `is_typing` inspects only the last `❯` line, so an absorbed Enter leaves a
  multi-line composer that reads as empty — the watcher then reports a
  successful submit that never happened. "The session went busy" is the only
  positive evidence.
- **Readiness ≠ the `❯` glyph** (still true): the glyph shows while the backend
  is `/rc connecting…`.
- **tmux socket is unreachable from the agent's sandboxed Bash**
  (`Operation not permitted`) — hook-spawned watchers are the only
  sandbox-clean path; spike driving needs `dangerouslyDisableSandbox`.
- **`claude --plugin-dir <path>`** (repeatable) loads a dev plugin tree into a
  throwaway session — the way to exercise a working-copy plugin in a scratch
  project without touching global enablement. Note the command registers
  unnamespaced (`/handoff`, not `/handoff:handoff`).

Empirical basis for [[project_precompact_drive]]. Sibling of
[[feedback_tmux_cc_popup]].
